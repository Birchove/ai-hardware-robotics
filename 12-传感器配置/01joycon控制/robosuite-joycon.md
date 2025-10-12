```

# create conda env
conda create -n robosuite python=3.10
conda activate robosuite

# install robosuite
git clone https://github.com/box2ai-robotics/robosuite-joycon.git
cd robosuite-joycon
pip3 install -r requirements.txt
pip3 install -r requirements-extra.txt

# install joycon-robotics
cd ..
git clone https://github.com/box2ai-robotics/joycon-robotics.git
cd joycon-robotics
pip install -e .
sudo apt install dkms
make install

---

cd ..
cd robosuite-joycon

# defualt for keyboard control
python robosuite/demos/demo_device_control.py 


sudo apt install libhidapi-dev
# Single joycon(R) controller
python robosuite/demos/demo_device_control_joycon_single.py

# Bimanual with two joycon controller
python robosuite/demos/demo_device_control_joycon_bimanual.py
```





```
sudo snap install bun-js
```







如果缺少相关依赖：

这个问题和上一个类似，都是**缺少了底层的系统依赖库**。

------



### **问题原因** 🧐



你看到的这个 `ImportError` 意味着 Python 的 `hid` 包（也叫 `hidapi`）无法在你的系统里找到它需要调用的核心库文件。

简单来说，这分两层：

1. **Python 包 (`hid`)**: 这是你在 `micromamba` 环境里安装的一个 Python 库。它本身不直接和硬件通信，而是作为一个“翻译官”或“遥控器”。
2. **系统库 (`libhidapi-\*.so`)**: 这是在你的操作系统层面安装的一个 C 语言库。它才是真正负责和 USB HID 设备（比如 Joy-Con）沟通的“引擎”。

报错信息 `Unable to load any of the following libraries: libhidapi-hidraw.so ...` 明确地告诉你，Python 这个“遥控器”找不到可以控制的“引擎”。

------



### **解决方案** 🛠️



你需要为你的操作系统安装这个缺失的 `hidapi` 引擎库。

**1. 安装 `hidapi` 开发库**

这个库的包名在不同 Linux 发行版上不一样。打开你的终端，执行以下命令：

- 对于 **Debian / Ubuntu** (以及 WSL 里的 Ubuntu):

  Bash

  ```
  sudo apt update
  sudo apt install libhidapi-dev
  ```

  *安装 `-dev` 版本通常会把运行库 (`libhidapi-hidraw0` 等) 作为依赖一起装上，并且包含了编译时可能需要的头文件，是最稳妥的选择。*

- 对于 **Arch Linux**:

  Bash

  ```
  sudo pacman -S hidapi
  ```

- 对于 **Fedora / CentOS**:

  Bash

  ```
  sudo dnf install hidapi-devel
  ```

**2. (重要) 配置 `udev` 规则**

安装好库之后，通常你还需要给 Joy-Con 设置一下设备权限，否则普通用户可能无法访问它，导致 Python 程序即使能找到库也连不上手柄。

- **创建 udev 规则文件**: 这条命令会为所有任天堂设备 (Vendor ID `057e`) 设置权限，让所有用户都能读写。

  Bash

  ```
  echo 'SUBSYSTEM=="hidraw", ATTRS{idVendor}=="057e", MODE="0666"' | sudo tee /etc/udev/rules.d/99-joycon.rules
  ```

- **重新加载 udev 规则**: 让规则立即生效。

  Bash

  ```
  sudo udevadm control --reload-rules
  sudo udevadm trigger
  ```

**3. 重新连接你的 Joy-Con** 🎮

断开 Joy-Con 的蓝牙连接，然后重新配对连接一次，以确保新的 `udev` 规则应用到它上面。

**4. 再次运行你的 Python 程序**

完成以上步骤后，回到你的 `micromamba` 环境，再次运行你的 Python 脚本，这次应该就能成功导入 `hid` 模块并找到设备了。

Bash

```
python /home/kewei/17robo/robosuite-joycon/robosuite/demos/demo_device_control_joycon_bimanual.py
```
