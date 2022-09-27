# 平民作画 AI 来袭，你也可以拥有

上次一次尝试 AI 作画，还是在 6 月份，详情可见 《[AI 作画初体验](https://mp.weixin.qq.com/s/-oVd2ZnId4BSGC2Q2emQYA)》。那个时候使用的是 Google 开发的 DD (Disco Diffusion) 系统，最新版本为 V5.2。DD 作画的确令人惊艳。没想到，不到两个月的时间，SD (Stable Diffusion) 斜里杀出，一下子抢了 DD 的风头。之前研究 DD


```
# 下载最新版本 Anaconda 3
$ wget https://repo.anaconda.com/archive/Anaconda3-2022.05-Linux-x86_64.sh
# 修改文件为可执行
$ chmod a+x Anaconda3-2022.05-Linux-x86_64.sh
# 安装 Anaconda 3
$ ./Anaconda3-2022.05-Linux-x86_64.sh
```

接下来会有提示，

```
Do you accept the license terms? [yes|no]
[no] >>> yes

Anaconda3 will now be installed into this location:
/home/alex/anaconda3

  - Press ENTER to confirm the location
  - Press CTRL-C to abort the installation
  - Or specify a different location below

[/home/alex/anaconda3] >>> 
```

```
done
installation finished.
Do you wish the installer to initialize Anaconda3
by running conda init? [yes|no]
[no] >>> yes
no change     /data/ai/anaconda3/condabin/conda
no change     /data/ai/anaconda3/bin/conda
no change     /data/ai/anaconda3/bin/conda-env
no change     /data/ai/anaconda3/bin/activate
no change     /data/ai/anaconda3/bin/deactivate
no change     /data/ai/anaconda3/etc/profile.d/conda.sh
no change     /data/ai/anaconda3/etc/fish/conf.d/conda.fish
no change     /data/ai/anaconda3/shell/condabin/Conda.psm1
no change     /data/ai/anaconda3/shell/condabin/conda-hook.ps1
no change     /data/ai/anaconda3/lib/python3.9/site-packages/xontrib/conda.xsh
no change     /data/ai/anaconda3/etc/profile.d/conda.csh
no change     /home/alex/.bashrc
No action taken.
If you'd prefer that conda's base environment not be activated on startup, 
   set the auto_activate_base parameter to false: 

conda config --set auto_activate_base false

Thank you for installing Anaconda3!

```




```
$ conda config --set auto_activate_base false
```


```
$ git clone https://github.com/invoke-ai/InvokeAI.git
$ cd InvokeAI/
```

```
$ mkdir -p models/ldm/stable-diffusion-v1/
$ ln -s /data/ai/gan/sd/sd-v1-4.ckpt models/ldm/stable-diffusion-v1/model.ckpt
```

```
$ conda activate ldm
(ldm) alex@alex-MS-7C22:/data/ai/gan/sd/InvokeAI$ python scripts/preload_models.py

```