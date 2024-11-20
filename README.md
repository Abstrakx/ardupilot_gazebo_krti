# Tutorial SITL + Ardupilot + Gazebo + ROS Camera plugin
SITL + ArduPilot + Gazebo + Plugin Kamera ROS (Antarmuka Simulasi Software In Loop, Model)

## Deskripsi:

Akhirnya sebuah tutorial lengkap untuk menyiapkan drone virtual menggunakan Ardupilot (Arducopter) + SITL dalam lingkungan virtual 3D lengkap yang disediakan oleh Gazebo.

Yang terakhir namun tidak kalah penting (Sebenarnya yang paling sulit untuk mencari contohnya...) ditambahkan plugin kamera dari ROS untuk mempublikasikan gambar sehingga Anda dapat mengambilnya di luar Gazebo dan melakukan sesuatu dengannya...

Berikut adalah persyaratan, langkah-langkah pengaturan, dan cara penggunaan masing-masing bagian.

## Persyaratan:
* Ubuntu (20.04 LTS) Sangat direkomendasikan grafis 3D penuh
* Gazebo versi 11.x
* ROS Noetic (Diperlukan untuk bekerja dengan Gazebo)
* MAVROS

## Gazebo 11:

### Atur komputer Anda untuk menerima perangkat lunak dari packages.osrfoundation.org
````
sudo sh -c 'echo "deb http://packages.osrfoundation.org/gazebo/ubuntu-stable `lsb_release -cs` main" > /etc/apt/sources.list.d/gazebo-stable.list'
````

### Pengaturan kunci
````
wget http://packages.osrfoundation.org/gazebo.key -O - | sudo apt-key add -
````

### Instalasi Gazebo
````
sudo apt-get update
sudo apt-get install gazebo11
sudo apt install libgazebo-dev 
````

### Uji Gazebo
````
gazebo --verbose
````

CATATAN: Saat menjalankan dalam mode VM, ini mungkin diperlukan agar gazebo dapat berjalan
````
export SVGA_VGPU10=0
````

Untuk membuat perubahan permanen gunakan:
````
$ echo "export SVGA_VGPU10=0" >> ~/.profile
````

## Instalasi ROS noetic:

Instal ROS dengan sudo apt install ros-noetic-desktop-full (ikuti instruksi di http://wiki.ros.org/noetic/Installation/Ubuntu).

### Konfigurasi repositori Ubuntu Anda
````
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
sudo apt-key adv --keyserver hkp://ha.pool.sks-keyservers.net:80 --recv-key 421C365BD9FF1F717815A3895523BAEEB01FA116

sudo apt update
````

### Instal ROS noetic
````
sudo apt install ros-noetic-desktop-full
````

### Inisialisasi ROS
````
sudo rosdep init
rosdep update
````

### Pengaturan lingkungan
````
echo "source /opt/ros/melodic/setup.bash" >> ~/.bashrc
source ~/.bashrc
````

### Dependensi untuk membangun paket
````
sudo apt install python-rosinstall python-rosinstall-generator python-wstool build-essential
````

## Instalasi MAVROS:

Node komunikasi MAVLink yang dapat diperluas untuk ROS dengan proxy untuk Ground Control Station (Lihat instruksi asli di http://ardupilot.org/dev/docs/ros-install.html#installing-mavros).

### Konfigurasi repositori Ubuntu Anda
````
sudo apt-get install ros-noetic-mavros ros-noetic-mavros-extras

cd ~/

wget https://raw.githubusercontent.com/mavlink/mavros/master/mavros/scripts/install_geographiclib_datasets.sh

chmod a+x install_geographiclib_datasets.sh

./install_geographiclib_datasets.sh
````

### Untuk kemudahan penggunaan pada komputer desktop, silakan instal juga RQT
````
sudo apt-get install ros-noetic-rqt ros-noetic-rqt-common-plugins ros-noetic-rqt-robot-plugins
````

### Instal catkin tools
````
sudo sh \
    -c 'echo "deb http://packages.ros.org/ros/ubuntu `lsb_release -sc` main" \
        > /etc/apt/sources.list.d/ros-latest.list'
wget http://packages.ros.org/ros.key -O - | sudo apt-key add -
sudo apt-get update
sudo apt-get install python3-catkin-tools
````

Sekarang setelah kita memiliki semua yang terinstal dengan benar, kita dapat memulai konfigurasi sistem

## Pengaturan dan konfigurasi lingkungan:
* LANGKAH 1 - SITL Ardupilot
* LANGKAH 2 - Plugin gazebo Ardupilot (Versi asli khancyr)
* LANGKAH 3 - Plugin Gazebo ROS (roscam)
* LANGKAH 4 - Menghubungkan ArduPilot ke ROS

## LANGKAH 1 - Instalasi SITL Ardupilot:

Instruksi diambil dari ardupilot.org (Lihat instruksi asli di http://ardupilot.org/dev/docs/setting-up-sitl-on-linux.html).

### Klon repositori ArduPilot
````
git clone --recurse-submodules https://github.com/ArduPilot/ardupilot.git
cd ardupilot
````

### Instal beberapa paket yang diperlukan

Jika Anda menggunakan sistem berbasis debian (seperti Ubuntu atau Mint), kami menyediakan skrip yang akan melakukannya untuk Anda. Dari direktori ardupilot:

````
Tools/environment_install/install-prereqs-ubuntu.sh -y
````

Muat ulang path (log-out dan log-in untuk membuat permanen):
````
. ~/.profile
````

Sekarang melakukan proses build pada SITL kita : 
````
./waf configure --board sitl
./waf copter 
````
Reboot perangkat sebelum menguji SITL yang sudah diinstall
````
sudo reboot
````

### Finalisasi dan uji instalasi

Untuk memulai simulator, pertama-tama pindah ke direktori kendaraan. Misalnya, untuk kode multicopter pindah ke ardupilot/ArduCopter:

````
cd ~/ardupilot/ArduCopter
````

Kemudian mulai simulator menggunakan sim_vehicle.py. Pertama kali Anda menjalankannya, Anda harus menggunakan opsi -w untuk menghapus EEPROM virtual dan memuat parameter default yang benar untuk kendaraan Anda.

````
sim_vehicle.py --console --map
````

### Memperbarui MAVProxy dan pymavlink

Versi baru MAVProxy dan pymavlink dirilis cukup teratur. Jika Anda adalah pengguna SITL reguler, Anda harus memperbarui sesekali menggunakan perintah ini

````
sudo pip install --upgrade pymavlink MAVProxy
````

Ini mengakhiri langkah pertama instalasi SITL Ardupilot.

## LANGKAH 2 - Instalasi plugin gazebo Ardupilot:

(Lihat instruksi asli di https://github.com/khancyr/ardupilot_gazebo).

### Klon repositori ArduPilot
````
cd ~/

git clone https://github.com/r0ch1n/ardupilot_gazebo

cd ardupilot_gazebo

mkdir build

cd build

cmake ..

# gunakan make tanpa parameter jika menjalankan di VM
make -j4

sudo make install
````

### Atur variabel lingkungan
````
echo 'source /usr/share/gazebo/setup.sh' >> ~/.bashrc
````

Atur Path Model Gazebo (Sesuaikan path ke tempat mengklon repo)
````
echo 'export GAZEBO_MODEL_PATH=~/ardupilot_gazebo/models' >> ~/.bashrc
````

Atur Path World Gazebo (Sesuaikan path ke tempat mengklon repo)
````
echo 'export GAZEBO_RESOURCE_PATH=~/ardupilot_gazebo/worlds:${GAZEBO_RESOURCE_PATH}' >> ~/.bashrc
````

Muat ulang path (log-out dan log-in untuk membuat permanen):
````
source ~/.bashrc
````

### Uji instalasi

Buka satu Terminal dan jalankan SITL Ardupilot
````
sim_vehicle.py -v ArduCopter -f gazebo-iris --map --console
````

Buka Terminal kedua dan jalankan Gazebo dengan plugin ardupilot_gazebo
````
gazebo --verbose worlds/iris_arducopter_runway.world
````

Anda seharusnya melihat dunia gazebo dengan quadcopter kecil tepat di tengah

## LANGKAH 3 - Plugin Gazebo ROS (roscam):

Ini berisi model terintegrasi ROS kustom dan file .world untuk Gazebo

### Klon repositori model terintegrasi Gazebo roscam
````
# Source ROS
source /opt/ros/melodic/setup.bash

# Klon paket Gazebo ROS kustom
cd ~/

git clone https://github.com/Abstrakx/ardupilot_gazebo_krti.git

cd ardupilot_gazebo_krti

catkin init

cd src

catkin_create_pkg ardupilot_gazebo

cd ..

catkin build

# Tambahkan model dan plugin Kustom ke Gazebo
export GAZEBO_MODEL_PATH=~/ardupilot_gazebo/models:$GAZEBO_MODEL_PATH
export GAZEBO_MODEL_PATH=~/ardupilot_gazebo_krti/src/ardupilot_gazebo/models:$GAZEBO_MODEL_PATH
export GAZEBO_PLUGIN_PATH=/usr/lib/x86_64-linux-gnu/gazebo-9/plugins:$GAZEBO_PLUGIN_PATH 
export GAZEBO_PLUGIN_PATH=/opt/ros/melodic/lib:$GAZEBO_PLUGIN_PATH

# Uji instalasi
source ~/ardupilot_gazebo_krti/devel/setup.bash

roslaunch ardupilot_gazebo iris_with_roscam.launch
````

## LANGKAH 4 - Menghubungkan ArduPilot ke ROS menggunakan MAVROS:

Hubungkan ke Ardupilot dari ROS (Ardupilot <–> MAVLink <–> ROS) Informasi asli diambil dari http://ardupilot.org/dev/docs/ros-sitl.html
Catatan - Gazebo tidak termasuk dalam MAVROS sehingga Anda tidak dapat terhubung atau mengakses Lingkungan Gazebo apa pun.

### Pengaturan MAVROS

Versi baru MAVProxy dan pymavlink dirilis cukup teratur. Jika Anda adalah pengguna SITL reguler, Anda harus memperbarui sesekali menggunakan perintah ini

````
cd ~/

mkdir -p ardupilot_ws/src

cd ardupilot_ws

catkin init

cd src

mkdir launch

cd launch

roscp mavros apm.launch apm.launch

sudo gedit apm.launch

Untuk terhubung ke SITL kita hanya perlu memodifikasi baris pertama menjadi <arg name="fcu_url" default="udp://127.0.0.1:14551@14555" />. simpan file Anda dan jalankan dengan
````

### Uji
````
cd ~/ardupilot_ws/src/launch

roslaunch apm.launch
````

## Menjalankan semuanya

### Jalankan Gazebo

Buka satu Terminal dan jalankan Gazebo terintegrasi ROS

````
#Pastikan Anda memiliki semua lingkungan yang benar, jika Anda tidak yakin jalankan yang berikut ini terlebih dahulu

source /opt/ros/melodic/setup.bash

export GAZEBO_MODEL_PATH=~/ardupilot_gazebo/models:$GAZEBO_MODEL_PATH
export GAZEBO_MODEL_PATH=~/ardupilot_gazebo_krti/src/ardupilot_gazebo/models:$GAZEBO_MODEL_PATH
export GAZEBO_PLUGIN_PATH=/usr/lib/x86_64-linux-gnu/gazebo-9/plugins:$GAZEBO_PLUGIN_PATH 
export GAZEBO_PLUGIN_PATH=/opt/ros/melodic/lib:$GAZEBO_PLUGIN_PATH

#Jalankan Gazebo terintegrasi ROS

source ~/ardupilot_gazebo_krti/devel/setup.bash

roslaunch ardupilot_gazebo krti.launch
````

### Jalankan SITL Ardupilot

Buka Terminal kedua dan jalankan SITL Ardupilot

````
cd ~/ardupilot/ArduCopter

sim_vehicle.py -f gazebo-iris --console --map
````
### Jalankan MAVROS Node

Buka Terminal ketiga dan jalankan SITL Ardupilot

````
roslaunch mavros apm.launch fcu_url:=udp://:14550@
````

### Subscribe topic feed roscam virtual

Buka Terminal keempat dan RTL

````
rqt
````

Pilih Plugins -> Visualization -> Image View

Kemudian pilih /roscam/cam/image_raw

Anda seharusnya melihat feed langsung dari dalam gazebo

