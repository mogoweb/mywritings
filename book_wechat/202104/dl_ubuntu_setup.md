
sudo apt-get install fcitx libqtwebkit4 libopencc2 fcitx-libs libfcitx-qt0


#### docker

https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-18-04

```
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
sudo apt update
```

```
alex@alex-MS-7C22:~$ sudo apt install docker-ce
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following additional packages will be installed:
  aufs-tools cgroupfs-mount containerd.io docker-ce-cli pigz
The following NEW packages will be installed:
  aufs-tools cgroupfs-mount containerd.io docker-ce docker-ce-cli pigz
0 upgraded, 6 newly installed, 0 to remove and 0 not upgraded.
Need to get 85.6 MB of archives.
After this operation, 384 MB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://cn.archive.ubuntu.com/ubuntu bionic/universe amd64 pigz amd64 2.4-1 [57.4 kB]
Get:2 https://download.docker.com/linux/ubuntu bionic/stable amd64 containerd.io amd64 1.2.10-3 [20.0 MB]
Get:3 http://cn.archive.ubuntu.com/ubuntu bionic/universe amd64 aufs-tools amd64 1:4.9+20170918-1ubuntu1 [104 kB]
Get:4 http://cn.archive.ubuntu.com/ubuntu bionic/universe amd64 cgroupfs-mount all 1.4 [6,320 B]
Get:5 https://download.docker.com/linux/ubuntu bionic/stable amd64 docker-ce-cli amd64 5:19.03.4~3-0~ubuntu-bionic [42.5 MB]                                                                               
Get:6 https://download.docker.com/linux/ubuntu bionic/stable amd64 docker-ce amd64 5:19.03.4~3-0~ubuntu-bionic [22.9 MB]                                                                                   
Fetched 85.6 MB in 39s (2,215 kB/s)                                                                                                                                                                        
Selecting previously unselected package pigz.
(Reading database ... 188404 files and directories currently installed.)
Preparing to unpack .../0-pigz_2.4-1_amd64.deb ...
Unpacking pigz (2.4-1) ...
Selecting previously unselected package aufs-tools.
Preparing to unpack .../1-aufs-tools_1%3a4.9+20170918-1ubuntu1_amd64.deb ...
Unpacking aufs-tools (1:4.9+20170918-1ubuntu1) ...
Selecting previously unselected package cgroupfs-mount.
Preparing to unpack .../2-cgroupfs-mount_1.4_all.deb ...
Unpacking cgroupfs-mount (1.4) ...
Selecting previously unselected package containerd.io.
Preparing to unpack .../3-containerd.io_1.2.10-3_amd64.deb ...
Unpacking containerd.io (1.2.10-3) ...
Selecting previously unselected package docker-ce-cli.
Preparing to unpack .../4-docker-ce-cli_5%3a19.03.4~3-0~ubuntu-bionic_amd64.deb ...
Unpacking docker-ce-cli (5:19.03.4~3-0~ubuntu-bionic) ...
Selecting previously unselected package docker-ce.
Preparing to unpack .../5-docker-ce_5%3a19.03.4~3-0~ubuntu-bionic_amd64.deb ...
Unpacking docker-ce (5:19.03.4~3-0~ubuntu-bionic) ...
Setting up aufs-tools (1:4.9+20170918-1ubuntu1) ...
Setting up containerd.io (1.2.10-3) ...
Created symlink /etc/systemd/system/multi-user.target.wants/containerd.service → /lib/systemd/system/containerd.service.
Setting up cgroupfs-mount (1.4) ...
Setting up docker-ce-cli (5:19.03.4~3-0~ubuntu-bionic) ...
Setting up pigz (2.4-1) ...
Setting up docker-ce (5:19.03.4~3-0~ubuntu-bionic) ...
Created symlink /etc/systemd/system/multi-user.target.wants/docker.service → /lib/systemd/system/docker.service.
Created symlink /etc/systemd/system/sockets.target.wants/docker.socket → /lib/systemd/system/docker.socket.
Job for docker.service failed because the control process exited with error code.
See "systemctl status docker.service" and "journalctl -xe" for details.
invoke-rc.d: initscript docker, action "start" failed.
● docker.service - Docker Application Container Engine
   Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
   Active: activating (auto-restart) (Result: exit-code) since Sat 2019-11-09 14:47:36 CST; 16ms ago
     Docs: https://docs.docker.com
  Process: 672 ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock (code=exited, status=1/FAILURE)
 Main PID: 672 (code=exited, status=1/FAILURE)
dpkg: error processing package docker-ce (--configure):
 installed docker-ce package post-installation script subprocess returned error exit status 1
Processing triggers for libc-bin (2.27-3ubuntu1) ...
Processing triggers for systemd (237-3ubuntu10.31) ...
Processing triggers for man-db (2.8.3-2ubuntu0.1) ...
Processing triggers for ureadahead (0.100.0-21) ...
ureadahead will be reprofiled on next reboot
Errors were encountered while processing:
 docker-ce
E: Sub-process /usr/bin/dpkg returned an error code (1)
alex@alex-MS-7C22:~$ sudo systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
   Active: failed (Result: exit-code) since Sat 2019-11-09 14:47:45 CST; 23s ago
     Docs: https://docs.docker.com
  Process: 3764 ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock (code=exited, status=1/FAILURE)
 Main PID: 3764 (code=exited, status=1/FAILURE)

11月 09 14:47:45 alex-MS-7C22 systemd[1]: docker.service: Service hold-off time over, scheduling restart.
11月 09 14:47:45 alex-MS-7C22 systemd[1]: docker.service: Scheduled restart job, restart counter is at 4.
11月 09 14:47:45 alex-MS-7C22 systemd[1]: Stopped Docker Application Container Engine.
11月 09 14:47:45 alex-MS-7C22 systemd[1]: docker.service: Start request repeated too quickly.
11月 09 14:47:45 alex-MS-7C22 systemd[1]: docker.service: Failed with result 'exit-code'.
11月 09 14:47:45 alex-MS-7C22 systemd[1]: Failed to start Docker Application Container Engine.
```

重启解决

Step 2 — Executing the Docker Command Without Sudo (Optional)

By default, the docker command can only be run the root user or by a user in the docker group, which is automatically created during Docker’s installation process. If you attempt to run the docker command without prefixing it with sudo or without being in the docker group, you’ll get an output like this:

```
docker: Cannot connect to the Docker daemon. Is the docker daemon running on this host?.
See 'docker run --help'.
```
If you want to avoid typing sudo whenever you run the docker command, add your username to the docker group:

```
sudo usermod -aG docker ${USER}
```

To apply the new group membership, log out of the server and back in, or type the following:

```
su - ${USER}
```
You will be prompted to enter your user’s password to continue.

Confirm that your user is now added to the docker group by typing:

id -nG

sammy sudo docker



https://nvidia.github.io/nvidia-docker/

docker run --gpus all nvidia/cuda:9.0-base nvidia-smi

CUDA 10.1

alex@alex-MS-7C22:~/NVIDIA_CUDA-10.1_Samples$ docker run --gpus all nvidia/cuda:9.0-base nvidia-smi
Unable to find image 'nvidia/cuda:9.0-base' locally
9.0-base: Pulling from nvidia/cuda
f7277927d38a: Pull complete 
8d3eac894db4: Pull complete 
edf72af6d627: Pull complete 
3e4f86211d23: Pull complete 
d6e9603ff777: Pull complete 
9454aa7cddfc: Pull complete 
a296dc1cdef1: Pull complete 
Digest: sha256:1883759ad42016faba1e063d6d86d5875cecf21c420a5c1c20c27c41e46dae44
Status: Downloaded newer image for nvidia/cuda:9.0-base
Sat Nov  9 08:58:47 2019       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 418.87.01    Driver Version: 418.87.01    CUDA Version: 10.1     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce RTX 208...  On   | 00000000:01:00.0  On |                  N/A |
| 27%   33C    P8    20W / 250W |    658MiB / 10986MiB |      6%      Default |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
+-----------------------------------------------------------------------------+

#### teamviewer

sudo apt-get install libqt5qml5 libqt5quick5 libqt5webkit5 libqt5x11extras5 qml-module-qtquick2 qml-module-qtquick-controls qml-module-qtquick-dialogs qml-module-qtquick-window2 qml-module-qtquick-layouts

sudo dpkg -i ~/Downloads/teamviewer_14.7.1965_amd64.deb

#### anaconda

conda create --name py2 python=2.7

docker:

https://medium.com/@mccode/understanding-how-uid-and-gid-work-in-docker-containers-c37a01d01cf

#### sublime text 3

wget -qO - https://download.sublimetext.com/sublimehq-pub.gpg | sudo apt-key add -

echo "deb https://download.sublimetext.com/ apt/stable/" | sudo tee /etc/apt/sources.list.d/sublime-text.list

sudo apt-get update
sudo apt-get install sublime-text

4.1 GBK编码支持
按ctrl+shit+p显示命令列表，在其中输入install，选择Package Control: Install package，待列表获取完成再输入GBK Encoding Support，等安装完毕就可以了。
按照上述方法安装Codecs33和ConvertToUTF8
菜单File下多了Reload with Encoding，接着选择Chinese Simplified (GBK)，中文不再是乱码了。

#### vs code

python
project manager

#### nginx

https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-18-04