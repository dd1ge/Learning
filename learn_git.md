# git用法学习

### 创建版本库，添加、提交文件

- 创建版本库 

  - 创建文件夹，文件夹中执行：

    ```bash
    git init
    ```

- 添加文件

  - 仓库内新建文件（可多个），再执行（一次执行）：

    ```bash
    git add file1 file2 ... filen
    ```

- 提交文件

  - 多次添加，一次提交，执行：

    ```bash
    git commit -m "描述"
    ```

### 版本间穿梭

- 查看工作区（当前仓库）的状态;

- 查看文件有无被修改过

  ```bash
  git status
  git diff
  ```

- 版本回退![image-20210403212935279](C:\Users\胡鉴\AppData\Roaming\Typora\typora-user-images\image-20210403212935279.png)



- 工作区与暂存区![git-repo](https://www.liaoxuefeng.com/files/attachments/919020037470528/0)