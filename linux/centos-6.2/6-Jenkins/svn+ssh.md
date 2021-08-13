###  Jenkins + SVN + ssh 配置

#### 1. 创建任务
- `Jenkins`->`New Item`，输入任务名：test，选择任务类型：Freestyle project，点击`OK`

#### 2. 配置任务
- `Jenkins`，点击要配置的任务：test，再点击`Configure`

- 配置`Source Code Management`->`Subversion`

  **Repository URL**
  ```
  svn+ssh://192.168.244.250/test/trunk
  ```
  **Credentials**
  ```
  使用os_svnuser用户名+os_dev3 ssh密钥，新建全局凭证: svn-auth-ssh
  ```
- 配置`Build`

  **Execute shell**
  ```sh
  bash build.sh
  ```
#### 3. 构建任务
- `Build Now`

