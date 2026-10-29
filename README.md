# Escola_de_agentes_tutorial

## 1. Instalar ferramentas de compilação

Atualize os índices e instale as ferramentas básicas usadas na compilação do rosbridge.

```bash
apt update
apt install --no-install-recommends git cmake build-essential
```

## 2. Instalar o colcon no ChonOS

O pacote `python3-colcon-common-extensions` não estava disponível; foi necessário instalar separadamente o núcleo, o executável e as extensões ROS/CMake.

```bash
apt install --no-install-recommends python3-colcon-core
apt install --no-install-recommends colcon
apt install --no-install-recommends python3-colcon-ros python3-colcon-cmake
colcon --help
```

## 3. Instalar as dependências Python do rosbridge

Instale os módulos utilizados pelo rosbridge e o Tornado fornecido pelo Debian.

```bash
apt install --no-install-recommends python3-bson python3-pil python3-cbor2 python3-tornado
python3 -c "import tornado; print(tornado.version, tornado.__file__)"
```

> Se `python3-cbor2` não estiver disponível no repositório, verifique a dependência indicada pelo erro de execução antes de escolher outro método de instalação.

## 4. Corrigir eventual conflito com Tornado antigo

Se o Python ainda carregar o Tornado 4.1 de `/usr/local/lib/python3.13/dist-packages`, remova somente essa instalação antiga feita com pip, pois ela impediu o handshake WebSocket em nossa primeira instalação.

```bash
python3 -m pip show tornado
python3 -m pip uninstall tornado --break-system-packages
python3 -c "import tornado; print(tornado.version, tornado.__file__)"
```

## 5. Clonar o rosbridge_suite

Crie um workspace separado e clone o repositório na branch `jazzy` utilizada na instalação.

```bash
mkdir -p ~/rosbridge_ws/src
cd ~/rosbridge_ws/src
git clone --depth 1 --single-branch -b jazzy https://github.com/RobotWebTools/rosbridge_suite.git
```

## 6. Compilar o rosbridge

Ignore o pacote de mensagens de teste, desative os testes CMake e compile o workspace.

```bash
touch ~/rosbridge_ws/src/rosbridge_suite/rosbridge_test_msgs/COLCON_IGNORE
cd ~/rosbridge_ws
source /opt/ros/jazzy/setup.bash
colcon build --symlink-install --cmake-args -DBUILD_TESTING=OFF
```

## 7. Teste do rosbridge

Carregue o ROS 2 e o workspace compilado e inicie o rosbridge na porta padrão `9090`.

```bash
source /opt/ros/jazzy/setup.bash
source ~/rosbridge_ws/install/setup.sh
ros2 run rosbridge_server rosbridge_websocket
```

A mensagem esperada é `Rosbridge WebSocket server started on port 9090`.

## 8. Clonar o embedded-mas

Em outro terminal, clone a branch `krepsky-devs`, que deve conter as correções de publicação das mensagens ROS 2.

```bash
cd ~
git clone --depth 1 --single-branch -b krepsky-devs https://github.com/embedded-mas/embedded-mas.git
```

testar:

```bash
cd ~/embedded-mas/temp/labra/ex1
./gradlew --no-daemon \
  -Dorg.gradle.jvmargs="-Xmx128m -XX:MaxMetaspaceSize=96m" \
  -Dorg.gradle.workers.max=1 \
  run
```

## 9. Clonar o javino_ros_bridge

1. Criar o workspace e entrar na pasta src

```bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src
```

2. Clonar o repositório

```bash
git clone --depth 1 --single-branch \
  -b javino_ros_bridge \
  https://github.com/KrepsD/Tutorial_ROS2.git
```

É preciso instalar a biblioteca do Javino em python antes de executar o nó javino_ros_bridge (e suas dependencias):

```bash
cd ~
git clone https://github.com/chon-group/javino2python.git
```

```bash
apt install --no-install-recommends python3-serial python3-setuptools
```

```bash
cd ~/javino2python
python3 -m pip install --break-system-packages --no-build-isolation .
```

Para verificar a instalação:

```bash
python3 -c "import javino; print(javino.__file__)"
```

Agora é possível rodar o nó:

```bash
source /opt/ros/jazzy/setup.bash
source ~/ros2_ws/install/setup.sh
ros2 run my_py_pkg javino_ros_bridge
```

## 10. Configurar a conexão ROS 2 no agente

No YAML do agente, utilize o endereço do rosbridge e configure os tópicos de percepção e ação.
