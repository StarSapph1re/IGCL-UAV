## cargo安装
sudo apt update
sudo apt install rustc cargo

## 源码编译pytorch
参考链接: https://forums.developer.nvidia.com/t/pytorch-for-jetson/72048  
nvidia官方在JetPack 5上只提供了python3.8对应的pytorch whl文件，需要重新编译  
首先需要确保cuda和cudnn都已经正确安装  
链接中展开build from source，按照里面的步骤进行即可，编译大约需要2.4小时...  

编译完成后，会在源码文件夹下dist/中生成whl文件  
```
conda activate <env_name>
pip install torch-<version>-cp39-cp39-linux_aarch64.whl
```

### - 编译2.1.0-cp39时出现 /torch.csrc/utils/tensor_numpy.cpp报错 no member named 'elsize'...

参考链接: https://github.com/pytorch/pytorch/issues/121798
```
// 404行左右，添加if判断
#if NPY_ABI_VERSION >= 0x02000000
    dtype_size_in_bytes = PyDataType_ELSIZE(descr);
#else
    dtype_size_in_bytes = descr->elsize;
#endif
    TORCH_INTERNAL_ASSERT(dtype_size_in_bytes > 0);
```

## flash-attention源码编译
### 遇到 No module named 'flash_attn_2_cuda'
参考链接：https://github.com/Dao-AILab/flash-attention/issues/933
aarch64平台上需要指定使用cuda，并从源码编译

```
git clone --recursive --brach v2.6.3 https://github.com/Dao-AILab/flash-attention.git
cd flash-attention
FLASH_ATTENTION_SKIP_CUDA_BUILD=FALSE
python setup.py install
```

### 源码编译内存不足 "killed"
如果是python setup.py install，编辑setup.py，里面会有设置make参数的，-j16改成-j8，-j4试试
