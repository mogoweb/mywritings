repo init -u ssh://alexzchen@gerrit.china-liantong.com:29000/venus/manifest -m smarttv_tvos3.xml
Get ssh://alexzchen@gerrit.china-liantong.com:29000/venus/manifest
Unable to negotiate with 10.27.254.101 port 29000: no matching cipher found. Their offer: aes128-cbc,3des-cbc,blowfish-cbc
Unable to negotiate with 10.27.254.101 port 29000: no matching cipher found. Their offer: aes128-cbc,3des-cbc,blowfish-cbc
fatal: Could not read from remote repository.

You can update your ssh configuration from the file located at: /etc/ssh/ssh_config

1. Launch a terminal.
2. Paste the line into the terminal: sudo nano /etc/ssh/ssh_config
3. Enter your password. Press Enter. SSH config file will be displayed.
4. Un-comment the line: Ciphers aes128-ctr,aes192-ctr,aes256-ctr,aes128-cbc,3des-cbc
5. Press Ctrl + X. Press Enter to save and exit.

Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?

sudo systemctl start docker.service

###　CUDA错误汇总

#### 1. cublas出错

版本：TensorFlow GPU 1.15、CUDA 10.0

**错误提示：**

```
2019-11-22 10:09:40.148427: E tensorflow/stream_executor/cuda/cuda_blas.cc:238] failed to create cublas handle: CUBLAS_STATUS_NOT_INITIALIZED
2019-11-22 10:09:40.148440: W tensorflow/stream_executor/stream.cc:2041] attempting to perform BLAS operation using StreamExecutor without BLAS support
Traceback (most recent call last):
  File "/data/ai/anaconda3/envs/tensorflow-gpu/lib/python3.6/site-packages/tensorflow_core/python/client/session.py", line 1365, in _do_call
    return fn(*args)
  File "/data/ai/anaconda3/envs/tensorflow-gpu/lib/python3.6/site-packages/tensorflow_core/python/client/session.py", line 1350, in _run_fn
    target_list, run_metadata)
  File "/data/ai/anaconda3/envs/tensorflow-gpu/lib/python3.6/site-packages/tensorflow_core/python/client/session.py", line 1443, in _call_tf_sessionrun
    run_metadata)
tensorflow.python.framework.errors_impl.InternalError: 2 root error(s) found.
  (0) Internal: Blas GEMM launch failed : a.shape=(100, 2048), b.shape=(2048, 120), m=100, n=120, k=2048
	 [[{{node final_retrain_ops/Wx_plus_b/MatMul}}]]
  (1) Internal: Blas GEMM launch failed : a.shape=(100, 2048), b.shape=(2048, 120), m=100, n=120, k=2048
	 [[{{node final_retrain_ops/Wx_plus_b/MatMul}}]]
	 [[final_retrain_ops/weights/summaries/stddev/Sqrt/_779]]
0 successful operations.
0 derived errors ignored.
```
**解决方法：**

```
rm -rf ~/.nv/
```

**原因：**

Found this suggestion in the NVIDIA developer forum: https://devtalk.nvidia.com/default/topic/1007071/cuda-setup-and-installation/cuda-error-when-running-matrixmulcublas-sample-ubuntu-16-04/post/5169223/

I suspect that during the driver update there where still some compiled files of the old version that were not compatible, or even that were corrupted during the process. 

sudo apt-get install texlive-latex-base texlive-fonts-recommended texlive-fonts-extra texlive-latex-extra
sudo apt-get install texlive-xetex

https://stackoverflow.com/questions/49482221/pandoc-markdown-to-pdf-image-position