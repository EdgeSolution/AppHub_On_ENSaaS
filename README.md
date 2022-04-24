# AppHub_On_EnSaaS
# 1.helm方式手动部署
## 首次部署
### 1. 环境准备
#### 1.1 安装kubectl

**参考步骤**

step1：终端运行下面命令获取kubectl的最新发行版
```
curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
```
step2：安装kubectl
```
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
chmod +x kubectl
mkdir -p ~/.local/bin/kubectl
mv ./kubectl ~/.local/bin/kubectl
```
setp3: 将kubectl加入环境变量
例如在/etc/profile末尾添加：`export PATH=.:~/.local/bin/kubectl:$PATH`
然后执行`Source /etc/profile`使其生效

setp4：最后通过`kubectl version –client`验证

#### 1.2 安装helm
以Ubuntu为例，官网网站，ubuntu下安装的方式为：

```
curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
sudo apt-get install apt-transport-https --yes
echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```


### 2. 订阅服务
进入EnSaaS4.0 Management Portal页面， 点击工作台中的服务中心，如下图

![图1-进入服务中心](https://user-images.githubusercontent.com/65381865/164954623-ff168449-4096-4d47-984a-fecf7c27798b.png)

#### 2.1 订阅RabbitMQ
订阅RabbitMQ服务并生成secret 

**step 1: 进入RabbitMQ服务密钥管理页面**

选择的RabbitMQ服务，点击密钥管理，进入其密钥管理页面，如下图

![图2-rabbitmq密钥管理页面](https://user-images.githubusercontent.com/65381865/164954622-da608efb-e6a1-4d69-b2ed-f0333bfe0edf.png)

**step 2: 创建RabbitMQ服务的密钥**

首先点击右上角“+”号，如图所示，开始创建

![图2-1-加号](https://user-images.githubusercontent.com/65381865/164954629-b844e94e-fffc-4b38-b2a4-fe8675a68915.png)

然后在如下图所示的弹出框中输入相应内容，即输入secret的名称，选择需要部署apphub的集群、工作空间和命名空间，点击确定，成功创建secret

![图2-2-创建密钥](https://user-images.githubusercontent.com/65381865/164954633-16ac48ec-c133-4ce1-89ac-082526d44142.png)

下图为example，可以共参考

![图2-3创建密钥demo](https://user-images.githubusercontent.com/65381865/164954639-336f3e57-b9f8-422f-9885-7ecf59ac4a9d.png)

创建成功后，在密钥管理列表中可以看到，如下图

![图2-4创建成功](https://user-images.githubusercontent.com/65381865/164954650-bea25ec3-f040-4d68-ada1-e786df4363a8.png)

#### 2.2 订阅 Postgresql DB服务

**step1：进Postgresql DB其密钥管理页面**

选择的Postgresql DB服务，点击密钥管理，进入其密钥管理页面，如下图

![图3-db服务进入密钥管理页面](https://user-images.githubusercontent.com/65381865/164954660-64c24689-1281-4c50-b01a-32550307caa9.png)

**step2：创建Postgresql DB服务的密钥**

首先同RabbitMQ一样点击右上角“+”号开始创建

**注意事项**

**1. 切记secret名称必须与2.1创建的RabbitMQ服务的secret名称保持一致**

**2. 添加参数(group: g_apphub)**


举例如下图所示

![图3-1-举例demo](https://user-images.githubusercontent.com/65381865/164954675-7db8d5d4-021f-4f1c-b900-adcbb3d88917.png)

创建成功后，会在密钥管理列表中可以看到，如下图

![图3-2创建成功](https://user-images.githubusercontent.com/65381865/164954678-6115a02a-fc5b-41cb-8ef8-4bdbad587e19.png)

### 3. 手动部署
#### 3.1 配置kubectl

**step1: 从Management Portal下载kubectl的confi档**

如下图所示位置，下载
![image](https://user-images.githubusercontent.com/65381865/164957973-6b6748c0-11b9-4b95-ab93-310fe8fa70de.png)


**step2：将下载的config档拷贝到本地**

将config文档拷贝到helm部署机的/root/.kube目录下，并重命名为config


#### 3.2 下载AppHub的helm包

**方法1：git clone（需要git环境）**

如果需要安装最新版本，则执行如下：

```
git clone https://github.com/EdgeSolution/AppHub_On_EnSaaS.git
```

如果需要安装指定版本，例如1.0.2，则执行如下：
```
git clone --branch 1.0.2 https://github.com/EdgeSolution/AppHub_On_EnSaaS.git
```

**方法2：网页端直接下载zip包**

通过网页打开https://github.com/EdgeSolution/AppHub_On_EnSaaS.git ，选择相应的tag版本，直接下载即可。



#### 3.3	修改values.yml文件中的相关配置
如果是git clone方式获取helm包，则进入路径修改values.yml
如果是下载的zip/tar.gz包，则解压后，进入路径修改values.yml

```
cd AppHub_On_EnSaaS/AppHub-HelmChart
vi values.yml
```

如下图：

![图4-value截图](https://user-images.githubusercontent.com/65381865/164954684-6e184971-1930-490e-a451-d6e1fba67f11.png)

其中,

1：secret name

2-4,6：与MP域名后缀一致

5，为站点名称

7：.命名空间名字和.集群名字组合

#### 3.4 执行helm install
在命令行终端AppHub-HelmChart目录，执行：
```
helm install apphub-manager –n $namespace .
```
其中，namespace为实际要部署的目标命令空间名称。

例如，本地demo部署命令如下： `helm install apphub-manager -n apphub .`

## imager版本升级部署

### 场景1：原部署环境仍然可用

如果原有的helm安装包文件都存在，则直接修改AppHub_On_EnSaaS/AppHub-HelmChart下的values.yml中image版本tag版本号，然后升级即可

**step1：修改value.yml**

```
cd AppHub_On_EnSaaS/AppHub-HelmChart
vi values.yml
```
参考Github上最新的value.yml中相应的image tag version修改。建议不要直接覆盖value.yml，否则还需要按照首次安装方式那样修改value.yml中global:部分的相关项。
仅修改docker images版本，如下图所示：

![image](https://user-images.githubusercontent.com/65381865/164960716-0b0b5b9b-b3ad-4d5f-9be5-3c9538d8cd65.png)

**step2: helm upgrade**

```
helm upgrade apphubmanager -n apphub .
```
### 场景2：使用新的部署环境升级部署

当原来helm部署环境不存在时，需要参考部分首次部署的步骤即：
#### 1.环境准备
参考首次安装的环境准备步骤

#### 2.helm upagrade
**step1: 配置kubectl config档，下载AppHub的helm包，修改value.yml**

参考首次安装的3.1-3.3

**step2：执行helm upgarde**

在命令行终端AppHub-HelmChart目录，执行：
```
helm upgrade apphub-manager –n $namespace .
```
其中，namespace为实际要部署的目标命令空间名称。

例如，本地demo部署命令如下： `helm upgrade apphub-manager -n apphub .`














