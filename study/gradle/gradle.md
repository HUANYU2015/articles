

install Command-Line Completion gradle-completion 
https://github.com/gradle/gradle-completion

``` groovy
public class ProjectVersion {
    private int major
    private int minor

    public ProjectVersion(int major, int minor) {
        this.major = major
        this.minor = minor
    }

    public int getMajor() {
        major
    }


    void setMajor(int major) {
        this.major = major
    }
}

ProjectVersion projectVersion = new ProjectVersion(1, 1)
println projectVersion.minor

ProjectVersion projectVersion1 = null

println projectVersion == projectVersion1

```
/Users/shuyunfeng/.ssh/gitlab_id_rsa    

``` groovy
// 1 可选的类型定义
def version = 1

// 2 assert
//assert version == 2

// 3 括号可选
println(version)
println version

// 4 字符串
def s1 = 'imooc'
def s2 = "imooc version is ${version}"
def s3 = '''my
name is
imooc'''

print(s1)
print(s2)
print(s3)

// 5 集合 api

// list
def builtools = ['ant', 'maven']
builtools << 'gradle'

assert builtools.getClass() == ArrayList
assert builtools.size() == 3

// map

def buildYears = ['ant':2000, 'maven':2009]
buildYears.gradle = 2010

print(buildYears.ant)
print(buildYears['gradle'])
print(buildYears.getClass())

// 闭包
```


``` bash
# GitHub.com
Host github.com
  Preferredauthentications publickey
  IdentityFile ~/.ssh/id_rsa

# GitLab.com
Host gitlab.com
  Preferredauthentications publickey
  IdentityFile ~/.ssh/gitlab_id_rsa
```

![Xnip2019-07-22_12-23-16](/assets/Xnip2019-07-22_12-23-16.png)