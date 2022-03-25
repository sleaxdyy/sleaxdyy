### **Khái niệm**
- Jenkins là một phần mềm tự động hóa, mã nguồn mở và viết bằng Java. Nó dùng để thực hiện chức năng tích hợp liên tục (gọi là CI – Continuous Integration) và xây dựng các tác vụ tự động hóa.

- Jenkins giúp tự động hóa các quy trình trong phát triển phần mềm, hiện nay được gọi theo thuật ngữ Tích hợp liên tục, và còn được dùng đến trong việc Phân phối liên tục. Jenkins là một phần mềm dạng server, chạy trên nền servlet với sự hỗ trợ của Apache Tomcat. Nó hỗ trợ hầu hết các phần mềm quản lý mã nguồn phổ biến hiện nay như Git, Subversion, Mercurial, ClearCase… Jenkins cũng hỗ trợ cả các mã lệnh của Shell và Windows Batch, đồng thời còn chạy được các mã lệnh của Apache Ant, Maven, Gradle… Người sáng tạo ra Jenkins là Kohsuke Kawaguchi. Phát hành theo giấy phép MIT nên Jenkins là phần mềm miễn phí.

- Việc kích hoạt build dự án phần mềm bằng Jenkins có thể được thực hiện bằng nhiều cách: dựa theo các lần commit trên mã nguồn, theo các khoảng thời gian, kích hoạt qua các URL, kích hoạt sau khi các job khác được build,…

- Ngoài ra các bạn có thể tìm hiểu thêm về Jenkins tại trang tài liệu chính thức của nó: https://www.jenkins.io/doc/
### **Cài đặt Jenkins trên Centos 7**
***Cài đặt Java***
```console
yum install java-1.8.0-openjdk-devel -y
```
***Tạo Jenkins repo***
```console
curl --silent --location http://pkg.jenkins-ci.org/redhat-stable/jenkins.repo | sudo tee /etc/yum.repos.d/jenkins.repo
```
***Thêm kho lưu trữ vào hệ thống***
```console
rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key
```
***Cài đặt Jenkins***
```console
yum install jenkins -y
```
***Kích hoạt chạy Jenkins khi khởi động hệ thống và bật Jenkins***
```console
systemctl enable jenkins
systemctl start jenkins
```
***Cấu hình Firewalld để mở cổng 8080***
```Console
firewall-cmd --add-port=8080/tcp --permanent
```
***Thiết lập Jenkins***

*Truy cập theo địa chỉ IP cùng với port 8080*

- Một màn hình tương tự như sau sẽ xuất hiện, nhắc bạn nhập mật khẩu Administrator được tạo trong quá trình cài đặt:

![image](https://cdn.discordapp.com/attachments/473366456092852246/956769077056114698/Screenshot_86-1.webp)

```console
cat /var/lib/jenkins/secrets/initialAdminPassword
```
- Sao chép mật khẩu, dán nó vào trường ô mật khẩu và nhấp vào Continue.

- Trên màn hình tiếp theo, bạn sẽ được hỏi xem bạn muốn cài đặt các plugin được đề xuất hay lựa chọn các plugin cụ thể. Nhấp vào Install suggested plugins được đề xuất và quá trình cài đặt sẽ bắt đầu ngay lập tức.
![image](https://cdn.discordapp.com/attachments/473366456092852246/956770665577467944/Screenshot_88-1.png)
- Tiếp theo thiết lập người dùng quản trị và URL
- Save and Finish
- Khởi động lại Jenkins
```console
systemctl restart jenkins
```
- Giao diện Jenkins
![image](https://cdn.discordapp.com/attachments/473366456092852246/956773312711753778/Screenshot_97-1.png)

### **Tích hợp Jenkins với Github**
**Cấu hình Github**
- Vào repo trong Github -> Chọn setting -> Webhook-> Add Webhook
- Nhập vào Payload URL : (Jenkins URL)/github-webhook/
![image](https://cdn.discordapp.com/attachments/473366456092852246/956814051609022464/jengit3.png)
- Add Webhook
**Cấu hình Jenkins**
- Chọn New Item -> Pipeline -> OK
- Tích chọn GitHub hook trigger for GITScm polling
![image](https://cdn.discordapp.com/attachments/473366456092852246/956815226005749801/Jenkins20Github20Hook20Trigger.png)
- Code Pineline
```console
pipeline {
    
    agent any
    stages {
        stage('Build') {
            steps{
                git url: '<URL Github Repo>'
            }
            
        }
    }
}
```
**Jenkins thông váo về Telegram**
- Mở Telegram, tìm BotFather (@BotFather)
- Gửi lệnh /newbot để tạo bot

- Vào tên bot (ví dụ: jenkinsits), sau đó vào username của bot (ví dụ: Jenkinsits_bot)

- Khi đó Jenkinsits_bot được tạo thành công và cấp mã token cho bot
![image](https://cdn.discordapp.com/attachments/473366456092852246/956820593876021258/telegram-bot.png)
- Thêm Bot vào Group cần thông báo hoặc tin nhắn riêng tư. - Lấy IDChat ID Bot 
```console
curl -i -X GET https://api.telegram.org/bot<BOT_TOKEN>/getUpdates
```
*Cấu hình Jenkins để gửi thông báo về telegram*

*Pipeline*
```console
curl -i -X GET "https://api.telegram.org/bot<BOT_TOKEN>/sendMessage?chat_id=<CHAT_ID>&text=<chat_text>"
```
- <BOT_TOKEN> = api token telegram bot
- <CHAT_ID> = chat_id vừa lấy được ở trên
- <chat_text> = tin nhắn bot sẽ gửi về tele
```console

pipeline {
    
    agent any
    stages {
        stage('Build') {
            steps{
                git url: '<URL Github Repo>'
            }
            
        }
    }
    post{
        success{
            sh 'curl -iX GET "https://api.telegram.org/bot<BOT_TOKEN>/sendMessage?chat_id=<CHAT_ID>&parse_mode=HTML&text=<b>Project</b> : jenkinsnew <b>Branch</b>: master <b>Build </b> : SUCCESS"'
        }
    
        failure{
            sh 'curl -iX GET "https://api.telegram.org/bot<BOT_TOKEN>/sendMessage?chat_id=<CHAT_ID>&parse_mode=HTML&text=<b>Project</b> : jenkinsnew <b>Branch</b>: master <b>Build </b> : FAILURE"'
        }
        
    }
}

```
