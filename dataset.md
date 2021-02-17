# 实验数据说明
实验项目代码在/home/users/houjyuf/FAS_SGTD_v2

实验用到的CASIA，Replay Attack数据集的处理及说明

首先得对数据集视频分帧，然后用mtcnn裁出脸部区域得到图像，接着用PRNet得到的对应的深度图。
注：与原始代码中是先通过检测脸的工具得到脸区域的四个值（左上角x,y坐标及高，宽），输入之前再进行随机裁剪/旋转/对比度变化等数据增强方法（后面代码会有说明）不同，
这里直接将裁好的脸送入到模型中。

CASIA位于服务器的/DISK3/houjyuf/dataset_old/sgtd_data_casia下
Replay位于服务器的/DISK3/houjyuf/dataset_old/sgtd_data_replay下
各自有6个子目录：
* ./train_images   训练集（裁脸后的图像，视作原图）
* ./train_depth    训练集（裁脸后的图像对应的深度图）
* ./test_images   测试集（裁脸后的图像）
* ./test_depth    测试集（裁脸后的图像对应的深度图）
* ./devel_images   开发（验证）集（裁脸后的图像）
* ./devel_depth    开发（验证）集（裁脸后的图像对应的深度图）

CASIA目录组织（以下代码是按照如下组织的目录进行处理，如果想按自己的方式处理，可以对原始数据集进行重新组织，修改后面讨论的代码）
6个子目录均是以下组织结构：

以./train_images/CASIA_real_1_1/1_scene.jpg为例，
路径第一部分表示这是训练集的原图；第二部分表示视频名，格式为 数据集名_标签_人员编号_拍摄设备，即CASIA数据集的真样本中1号人员的采样自1情形的视频；
第三部分表示视频中的帧号，即第1帧,scene表示原图，depth1D则表示深度图

* CASIA数据集包含50个实验人员，各有12段视频共600段视频（官方发布的20个做训练集，30个做测试集，但为了符合原始代码对开发集的要求，去了测试集后5个做开发集）
* 其中12段视频中，分别采集自12种不同的情形（即3个设备×4种真假情形），其中真人采集自1，2，HR_1，攻击样本采集又可分为三种，3，6，HR_2是纸张攻击，4，7，HR_3是挖孔的纸张攻击，5，8，HR_4是视频重放攻击。 

Replay目录组织与CASIA的同理，不同的是第二部分：
以./train_images//1.jpg
第二部分表示视频名，即Replay数据集的攻击样本中固定拍摄，采集自001号人员在01场景，用高分辨率的照片在adverse光照下的攻击的视频。
真样本的格式是 数据集名_real_real_人员编号_场景_拍摄设备_authenticate_环境光_序列号，例如
Replay_real_real_client117_session01_webcam_authenticate_adverse_2
假样本的格式是 数据集名_attack_手持/固定拍摄_攻击呈现设备_人员编号_场景_拍摄设备_攻击方式_环境光，例如
Replay_attack_fixed_highdef_client001_session01_highdef_photo_adverse

* Replay attack数据集包含50个实验人员，各有24段视频共1200段视频（15个做训练集，15个做开发集，20个做测试集）
* 24段视频中，包括4段真人，20段攻击视频
* 真人视频分别在adverse/controlled两种环境光各拍摄两段视频
* 攻击视频包含用高分辨率屏幕或手机进行重放攻击各8段（手持/固定在adverse/controlled下重放photo或video）
，打印攻击4段（手持/固定在adverse/controlled下采集）

需要在FLAGS.py定义数据集位置
```python
from easydict import EasyDict as edict

flags=edict()
path_gen_save = './model_save_Casia/'

src_path = '/DISK3/houjyuf/dataset_old/sgtd_data_casia'
flags.path.train_file=[src_path+'/train_images',
                      src_path+'/train_depth']
flags.path.dev_file=[src_path+'/devel_images',
                    src_path+'/devel_depth']
flags.path.test_file=[src_path+'/test_images',
                     src_path+'/test_depth']
```
# 数据预处理代码分析

---
以下分析的fas_sgtd_single_frame的代码分析，原始代码主要是针对Oulu数据集的实验，个人修改了部分函数及类以训练CASIA和Replay，以训练CASIA为例
## 训练数据处理
训练命令：(默认训练是CASIA训练集)
>python train.py

训练的脚本是train.py，首先调用input_fn_maker函数
```python
import tensorflow as tf
import FLAGS
from generate_data_train import input_fn_maker
from generate_network import generate_network as model_fn

flags=FLAGS.flags # setting paras
# log info setting
tf.logging.set_verbosity(tf.logging.INFO)
# data fn
# 注意在FLAGS.py定义中，train_list是包括原图和深度图两个路径的list类型，train_data_list就是一个二维list类型
train_data_list=[flags.path.train_file]   
train_input_fn = input_fn_maker(train_data_list, shuffle=True, 
                                batch_size=flags.paras.batch_size_train,
                                epoch=flags.paras.epoch)
```
input_fn_maker的函数实现在generate_data_train.py中
```python
suffix1='scene.jpg'  # 原图的帧的后缀
suffix2='depth1D.jpg' # 深度图的帧的后缀
suffix3='scene.dat'  # 保存未裁剪的图的脸部区域四个值信息的文件的后缀
fix_len = flags.paras.fix_len # 16

def input_fn_maker(train_list, shuffle=True, batch_size=None, epoch=1, padding_info=None):
    # step 1: 传入train_list创建类InputFnGenerator的实例
    GEN_OBJ = InputFnGenerator(train_list)
    print('InputFnGenerator has been obtained')
    def input_fn():
        # step 2.2: 调用input_fn_handle函数，随后调用GEN_OBJ的input_fn_generator方法（参见后面代码实现）
        def input_fn_handle():
            return GEN_OBJ.input_fn_generator(shuffle)
        # step 2.1: 生成一个Dataset的生成器方法，函数为input_fn_handle，设置相应的函数原型传入参数类型。
        ## Dataset.from_generator可以使用普通编程语言编写的外部子函数生成Dataset，这样几乎不受tensorflow编程不便的影响
        ds=Dataset.from_generator(input_fn_handle, \
                     (tf.string, tf.string, tf.string, tf.string, tf.int32, tf.string)
                     )
        if (flags.paras.prefetch>1):
            ds=ds.prefetch(flags.paras.prefetch)
        # step 2.3: 将ds对象的参数映射到parser_fun函数中，调用parser_fun()进行打包解析
        ds=ds.map(parser_fun, num_parallel_calls=16)
        # step 2.4: 划分批次，构成可送入网络迭代器
        if (shuffle):
            ds=ds.shuffle(buffer_size=flags.paras.shuffle_buffer)
        if (padding_info):
            ds.padded_batch(batch_size, padded_shapes=padding_info)
        else:
            ds=ds.batch(batch_size)
        ds=ds.repeat(epoch)

        value = ds.make_one_shot_iterator().get_next()
        return value
    # step 2:调用input_fn()
    return input_fn
```
InputFnGenerator的实现：
```python
class InputFnGenerator:
    def __init__(self, train_list):
        # 根据深度图找到是否存在对应的原图
        def find_path_scene(path_depthmap):
            path_gen_scene = []
            path_gen_depthmap, name_pure=os.path.split(path_depthmap)
            for path_list in train_list:
                #print(path_list[1], path_gen_depthmap)
                if path_list[1] == path_gen_depthmap:
                    path_gen_scene = path_list[0]
            if path_gen_scene == []:
                print('Can\'t find correct path scene')
                exit(1)
            path_scene = os.path.join(path_gen_scene, name_pure)        
            return path_scene
        
        if(not type(train_list)==list):
            raise NameError
        FILES_LIST=[]
        for fInd in range(len(train_list)):
            path_train_file=train_list[fInd]
            # step 1.1: 搜索出目录下的所有深度图的视频，path_train_file[1]是深度图路径
            FILES=glob.glob(os.path.join(path_train_file[1],'*'))
            FILES_LIST=FILES_LIST+FILES  # 把所有用于训练的数据集视频放到FILES_LIST中
    
        ## select protocol of IJCB， IJCB是训练Oulu定义的类
        # data_object = IJCB(flags.dataset.protocal, 'train')
        # step 1.2: 创建类Casia实例来检查FILES_LIST是否符合命名规范（如果不区分训练、测试时可跳过）
        data_object = Casia('train')   # Casia类的实现位于util/util_dataset.py中
        FILES_LIST = data_object.dataset_process(FILES_LIST)

        self.existFaceLists_all = []
        for i in range(len(FILES_LIST)):
            path_image = FILES_LIST[i]
            # step 1.3: 找到深度图对应的原图, 并搜索出所有深度图的帧，放到IMAGES中
            path_scene = find_path_scene(path_image)
            name_pure=os.path.split(path_image)[-1]
            
            IMAGES=glob.glob(os.path.join(path_image,'*'+suffix2))
            if IMAGES == []:
                continue
            # step 1.4: 生成文件列表，generate_existFaceLists_perfile定义在后面
            existFaceLists=generate_existFaceLists_perfile(name_pure, path_scene, IMAGES)
            self.existFaceLists_all += existFaceLists
        

    def input_fn_generator(self, shuffle):
        # step 2.2.1: 根据shffle是否为True来打乱existFaceList_all的图像
        if shuffle:
            random.shuffle(self.existFaceLists_all)

        for existList in self.existFaceLists_all:
            # step 2.2.2: 取出existList的元素，组成ALLDATA元组，以便送入到后续的处理函数
            [name_pure, path_image, path_scene, frame_ind, label, face_name_full]=existList
            ALLDATA=[name_pure.encode(), path_image.encode(), path_scene.encode(), frame_ind.encode(), label, face_name_full.encode()]
            # encode()是将string编码成bytes
            yield tuple(ALLDATA)
```

Casia类的实现位于util/util_dataset.py中
```python
## util/util_dataset.py
class Casia:
    def __init__(self, mode): 
        self.mode = mode
    # 验证视频命名是否符合CASIA的结构规范，此处可按自己方式修改，主要是为了后面训练查找文件方便
    def isInPotocol(self, file_name_full):
        file_name = os.path.split(file_name_full)[-1]
        name_split = file_name.split('_')
        if name_split[0] == 'CASIA':
            return True
        else:
            return False
    # 返回所有符合命名规范的视频
    def dataset_process(self, file_list):
        res_list = []
        for i in range(len(file_list)):
            file_name_full = file_list[i]
            if self.isInPotocol(file_name_full):
                res_list.append(file_name_full)
        print('Dataset Info:')
        print('----------------------------------------')
        print('CASIA', self.mode)
        print('File Counts:', len(res_list))
        print('----------------------------------------')

        return res_list
```

generate_existFaceLists_perfile的实现（这个函数要根据自己的输入修改）
在原始代码中，为了数据均衡需要对攻击样本做采样，但事先个人已手动做了数据均衡，就不需要这部分处理勒
```python
def generate_existFaceLists_perfile(name_pure, path_scene, IMAGES):
    '''
    name_pure: pure name of each video
    IMAGES: image(frame) list of each video
    return: lists of [path_image, start_ind, end_ind, label, face_name_full]
    '''
    res_list=[]

    stride_seq= 1 #flags.paras.stride_seq
    # path_image是帧所在的视频的路径
    path_image=IMAGES[0][:-len(os.path.split(IMAGES[0])[-1])]
    # label_name是帧所在的视频名
    label_name=name_pure.split('_')[1]
    # 标签所在的位置是由结构决定
    if (name_pure.split('_')[0]=='CASIA'):
        if(label_name=='attack'): # casia and replayAttack
            label=2
            # stride_seq *= 5  # down sampling for negative samples
        elif(label_name=='real'):
            label=1
    elif (name_pure.split('_')[0]=='ReplayAttack'):
        if (label_name == 'attack'):  # casia and replayAttack
            label = 2
        elif (label_name == 'real'):
            label = 1
    else: # ijcb train and dev, Oulu
        label=int(name_pure.split('_')[-1])

        if(label>=2 and label<=3): # down sampling for negative samples
            stride_seq *= 1
        if(label>=4 and label<=5): # down sampling for negative samples
            stride_seq *= 1
    
    if num_classes == 2:
        label=1 if label==1 else 2
    # modify the label: attack 1 real 0
    label = label - 1

    frame_ind = ''
    for i, image in enumerate(IMAGES):
        ind = image.split('/')[-1].split('_')[0]
        res_list.append([name_pure, path_image, path_scene, ind, label, image])
    # 每隔fix_len的长度采样res_list
    res_list = get_res_list(res_list, label)
    return res_list
```
get_res_list函数
```python
def get_res_list(res_list, label):
    if label == 0:
        fix_len_this = fix_len
    else:
        fix_len_this = int(fix_len/4)
    
    len_list = len(res_list)

    each_len = int(len_list/fix_len_this)
    res_list_new = []
    for i in range(0, len_list, each_len):
        res_list_new.append(res_list[i])
    return res_list_new
```

parser_fun()函数实现
```python
def parser_fun(name_pure, path_image, path_scene, frame_ind, label, face_name_full):
    # 传入相应参数到read_data_decode()，返回裁剪连接后的图像值： ALLDATA=[image_face_cat, vertices_map_cat, mask_cat]
    ALLDATA = tf.py_func(read_data_decode,
                         [name_pure, path_image, path_scene,frame_ind, label, face_name_full],
                         [tf.float32, tf.float32, tf.float32]
                         )

    features = {}

    features['images'] = tf.reshape(ALLDATA[0], padding_info['images']) / 255.0
    features['maps'] = tf.reshape(ALLDATA[1], padding_info['maps'])
    features['masks'] = tf.reshape(ALLDATA[2], padding_info['masks'])
    features['labels'] = tf.reshape(label, padding_info['labels'])
    features['names'] = tf.reshape(tf.cast(name_pure, tf.string), [1])
    return features
```
read_data_decode()函数实现,含随机裁剪数据增强的代码（已注释掉）
```python
def read_data_decode(name_pure, path_image, path_scene, frame_ind, label, face_name_full):
    name_pure = name_pure.decode()
    path_image = path_image.decode()
    path_scene = path_scene.decode()
    frame_ind = frame_ind.decode()
    face_name_full=face_name_full.decode()
    # CASIA video:  CASIA/train/real/subject/device
    # Replay video: Replay/train/real/subject_device, Replay/train/attack/fixed/subject_device
    image_face_list = []
    vertices_map_list = []
    
    i = frame_ind
    
    scene_name_full = os.path.join(path_scene, i +'_'+suffix1)
    mesh_name_full = os.path.join(path_image, i +'_'+suffix2)
    # get face position information
    # face_dat_name = os.path.join(path_scene, str(i) +'_'+suffix3)

    # image = Image.open(scene_name_full)
    image_face = Image.open(scene_name_full)
    # face_info = get_face_info(image, face_dat_name)

    # image_face = crop_face_from_scene(image, face_info, is_depth = False)
    image_face = image_face.resize([padding_info['images'][0], padding_info['images'][1]])
    image_face = np.array(image_face, np.float32) - 127.5

    # depth1d = Image.open(mesh_name_full)
    depth1d_face = Image.open(mesh_name_full)
    # depth1d_face = crop_face_from_scene(depth1d, face_info, is_depth = True)
    depth1d_face = depth1d_face.resize([padding_info['maps'][0], padding_info['maps'][1]])
    vertices_map = np.array(depth1d_face, np.float32)
    vertices_map = np.expand_dims(vertices_map, axis=0)
    vertices_map = np.expand_dims(vertices_map, axis=-1)
    image_face_list.append(image_face)
    vertices_map_list.append(vertices_map)

    image_face_cat = np.concatenate(image_face_list, axis=-1)
    vertices_map_cat = np.concatenate(vertices_map_list, axis=-1)
    mask_cat = np.array(vertices_map_cat > 0.0, np.float32)
    if not label == 0:
        vertices_map_cat = np.zeros(vertices_map_cat.shape, dtype=np.float32)

    #print(label, image_face_cat.shape, vertices_map_cat.shape, mask_cat.shape)
    ALLDATA=[image_face_cat, vertices_map_cat, mask_cat]
    #print(np.concatenate(vertices_map_list, axis=-1).shape)

    return ALLDATA

```

# 测试代码分析
测试命令：(默认测试用CASIA测试集，即CASIA的库内实验)
>python test.py

测试数据的预处理和train类似，可参考generate_data_test.py的实现。
测试代码的主函数是test.py
有online（默认）和offline两种模式，分别是在训练过程中在线测试，以及传入训练好的模型离线测试。
```python
if __name__ == '__main__':
    if isOnline:
        online_eval()
    else:
        offline_eval()
```
下面以online_eval()为例。
```python
def online_eval():
    # 读取最近的检查点
    all_path_ckpt = os.path.join(flags.path.model, 'checkpoint')
    iter_before = 0
    while True:
        time.sleep(interval_time)
        if not os.path.exists(all_path_ckpt): 
            continue
        fid = open(all_path_ckpt, 'r')
        lines = fid.readlines()
        fid.close()
        iter_now = int(lines[0].split('-')[-1][:-2])
        # 每间隔若干个迭代测试一次
        if iter_now - iter_before >= interval_iteration:
            # step 1: 测试
            officialEval(os.path.join(flags.path.model, 'model.ckpt-%d'%(iter_now)))
            # step 2: 记录
            getExel(path_txt, iter_now)
            iter_before = iter_now
```
officailEval函数，通过officialEvalSub()对开发集和验证集进行测试
```python
def officialEval(path_model_now):
    path_txt_dev = os.path.join(path_txt, 'Dev_scores.txt')
    path_txt_test = os.path.join(path_txt, 'Test_scores.txt')
    
    officialEvalSub(path_txt_dev, [flags.path.dev_file], 'dev', path_model_now)
    officialEvalSub(path_txt_test, [flags.path.test_file], 'test', path_model_now)   
```

officialEvalSub函数
```python
def officialEvalSub(txt_name, data_list, mode, path_model_now):
    def realProb(logits):
        #return np.exp(logits[1])/(np.exp(logits[0])+np.exp(logits[1]))
        x = np.array(logits)
        y = np.exp(x[0])/np.sum(np.exp(x))
        #y = x[0]
        return y
    def name_encode(name_):
        if mode == 'dev':
            return name_
        elif mode == 'test':
            # 源代码需要编码成十六进制，此部分可直接return name_
            return name_
            # name_split = name_.split('_')
            # name_10 = name_split[0] + name_split[3] + name_split[1] + name_split[2]
            # name_16 = hex(int(name_10))
            # name_16 = name_16[0] + name_16[2:]
            # return name_16
        else:
            print('Error mode: requires dev or test')
            exit(1)        

    eval_input_fn = input_fn_maker(data_list, shuffle=False, 
                                batch_size = 1,
                                epoch=1)
    features = fas_classifier.predict(
            input_fn=eval_input_fn,
            checkpoint_path=path_model_now)
    fid = open(txt_name, 'w')
    fea_ind = 0
    acc_mean = 0.0
    video_name = None
    video_score = 0.0
    video_frame_count = 0.0
    for feature in features:
        logits=feature['logits']
        '''
        logits_tmp = logits[1]
        logits[1] = logits[0]
        logits[0] = logits_tmp
        '''
        labels=feature['labels']
        names=feature['names']
        depth_map=feature['depth_map']   
        masks=feature['masks']  
        depth_map = depth_map[..., 0]*masks[..., 0]   
        
        depth_mean = np.sum(depth_map) / np.sum(masks[..., 0])
        logits[0] = depth_mean
        logits[1] = 1.0 - depth_mean

        out = np.argmax(np.array(logits))
        #out = 1 if out == 0 else 0
        acc = int(out == labels[0])
        acc_mean += float(acc)
        print(fea_ind, logits, out, labels, [name.decode() for name in names], acc)

        if (video_name == None):
            video_name = names[0].decode()
            video_score += realProb(logits)
            video_frame_count += 1.0
        elif (not names[0].decode() == video_name):
            video_score_mean = video_score/video_frame_count
            video_name_encode = name_encode(video_name)
            fid.write(video_name_encode + ',' + str(video_score_mean) + '\n')

            video_name = names[0].decode()
            video_score = 0.0
            video_frame_count = 0.0
            video_score += realProb(logits)
            video_frame_count += 1.0
        else:
            video_score += realProb(logits)
            video_frame_count += 1.0
        #save_contents(fea_ind, logits, labels, [name.decode() for name in names])
        fea_ind+=1
    video_score_mean = video_score/video_frame_count
    video_name_encode = name_encode(video_name)
    fid.write(video_name_encode + ',' + str(video_score_mean) + '\n')

    print('acc_mean:', acc_mean/float(fea_ind))
    fid.close()
```

get_Exel函数
```python
def getExel(path_scores, iter_now):
    Performances_this = util_OULU.get_scores_Protocol_1(os.path.split(path_scores)[0])
    scores_txt = os.path.join(path_scores, 'eval.txt')
    if not os.path.exists(scores_txt):
        lines = []
    else:
        fid = open(scores_txt, 'r')
        lines = fid.readlines()
        fid.close()
    str_list = [('%-7.4f'%(x)) for x in Performances_this]
    str_per = ''
    for str_ in str_list:
        str_per += str_ + ' '
    fid = open(scores_txt, 'w')
    line_new =  str_per + str(iter_now) + '\n'
    lines.append(line_new)
    fid.writelines(lines)
    fid.close()
```

## Prerequisite

* Python 3.6 (numpy, skimage, scipy)

* TensorFlow >= 1.4

* opencv2

* Pillow (PIL)

* easydict
