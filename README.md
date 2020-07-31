# Linux无界面自动化打卡组合拳

   适用于Linux系统的苏大每日健康打卡的无界面打卡程序，按照下面操作自行设置即可，主代码修改自[
   EuGEne](https://blog.csdn.net/weixin_44739303/article/details/106245788)老哥，谢谢你

   如果是macOS或者Windows可以魔改本教程实现打卡

   ## Python pip安装selenium

   ```bash
   pip install selenium
   ```

   ## CentOS 或其他版本Linux安装chrome

   ```bash
   curl https://intoli.com/install-google-chrome.sh | bash
   ```

   ## 如果是macOS或windows

   自己手动下载chrome浏览器就行，如果早就安装了就不用管了，然后可以参考原作者的实现有界面打卡

   ## 下载Chrome-driver

   先查看chrome版本

   ```
   google-chrome --version
   ```

   再下载对应版本的driver

   [http://npm.taobao.org/mirrors/chromedriver](http://npm.taobao.org/mirrors/chromedriver)上查地址

   复制一下链接地址

   然后拼接并解压

   ```bash
   wget http://npm.taobao.org/mirrors/chromedriver/84.0.4147.30/chromedriver_linux64.zip
   unzip chromedriver_linux64.zip
   ```

   ## 放入主程序

   新建一个`daka.py`的文件，这个地方修改了原作者的一些代码，主要在打开浏览器函数中作了修改

   ```python
   # -*- coding: utf-8 -*-
    
   from selenium import webdriver
   import time
   import json
   
   # 开启浏览器
   def openChrome():
       options = webdriver.ChromeOptions()
       options.add_argument('--no-sandbox') # 解决DevToolsActivePort文件不存在的报错
       options.add_argument('window-size=1600x900') # 指定浏览器分辨率
       options.add_argument('--disable-gpu') # 谷歌文档提到需要加上这个属性来规避bug
       options.add_argument('--hide-scrollbars') # 隐藏滚动条, 应对一些特殊页面
       options.add_argument('blink-settings=imagesEnabled=false') # 不加载图片, 提升速度
       options.add_argument('--headless') # 浏览器不提供可视化页面.linux下如果系统不支持可视化不加这条会启动失败
       options.add_argument('disable-infobars')
       driver = webdriver.Chrome(options=options,executable_path='./chromedriver')
       return driver
   
   def find(driver,str):
       try:
           driver.find_element_by_xpath(str)
       except:
           return False
       else:
           return True
   
   # 流程
   def operate_dk(driver):
       # 打开配置文件
       try:
           with open("config.json",encoding='utf-8') as f:
               config=json.load(f)
       except:
           print("配置文件错误！请检查配置文件是否与py文件放置于同一目录下，或配置文件是否出错！\n请注意'现人员位置'只可填入以下四项中的任意一项：'留校', '在苏州', '江苏省内其他地区', '在其他地区'")
           return -1
       else:
           url = "https://auth.suda.edu.cn/cas/login?service=https%3A%2F%2Fauth.suda.edu.cn%2Fsso%2Flogin%3Fredirect_uri%3Dhttp%253A%252F%252Fauth.suda.edu.cn%252Fsso%252Foauth2%252Fauthorize%253Fscope%253Dclient%2526response_type%253Dcode%2526state%253D123%2526redirect_uri%253Dhttps%25253A%25252F%25252Fauth.suda.edu.cn%25252Fmiddleware%25252Fclient%25253Fapp%25253Dsswsswzx2%252526welcome%25253D%2525E4%2525BA%25258B%2525E5%25258A%2525A1%2525E4%2525B8%2525AD%2525E5%2525BF%252583%252526x_app_url%25253Dhttp%25253A%25252F%25252Faff.suda.edu.cn%25252F_web%25252Fucenter%25252Flogin.jsp%2526client_id%253Dhuc7ES9iOtrLLhTkCZVk%26x_client%3Dcas"
        	 driver.get(url)
        	 # 找到输入框并输入查询内容
        	 elem = driver.find_element_by_id("username")
        	 elem.send_keys(config["学号"])
        	 elem = driver.find_element_by_id("password")
        	 elem.send_keys(config["密码"])
        	 # 提交表单
        	 driver.find_element_by_xpath("//*[@id='fm1']/div[7]").click()
           try:
               driver.find_element_by_xpath("//*[text()='首页']")
           except:
               print('登陆失败！请检查学号密码是否正确')
               return -1
           else:
               print('登录成功！')
               url = "http://aff.suda.edu.cn/_web/fusionportal/detail.jsp?_p=YXM9MSZwPTEmbT1OJg__&id=2749&entranceUrl=http%3A%2F%2Fdk.suda.edu.cn%2Fdefault%2Fwork%2Fsuda%2Fjkxxtb%2Fjkxxcj.jsp&appKey=com.sudytech.suda.xxhjsyglzx.jkxxcj."
               driver.get(url)
               driver.find_element_by_xpath("//*[@class='action-btn action-do']").click()
   
               # 切换到小框内
               iframe1=driver.find_element_by_id("layui-layer-iframe1")
               driver.switch_to.frame(iframe1)
               flag=False
               while(flag==False):
                   time.sleep(1)
                   flag=find(driver, "//*[text()='"+config["学号"]+"']")
   
               driver.find_element_by_xpath("//*[@id='checkbox_jkzk259']").click()
               xrywz={'留校':'radio_xrywz5','在苏州':'radio_xrywz7','江苏省内其他地区':'radio_xrywz9','在其他地区':'radio_xrywz37'}
               driver.find_element_by_xpath("//*[@id='"+xrywz[config["现人员位置"]]+"']").click()
               elem = driver.find_element_by_id("input_jtdz")
               elem.send_keys(config["家庭地址"])
               driver.find_element_by_xpath("//*[@id='radio_sfyxglz41']").click()
   
               # 提交
               driver.find_element_by_xpath("//*[@id='tpost']").click()
               time.sleep(1)
               try:
                   driver.find_element_by_xpath("//*[text()='提交成功！']")
                   print('打卡成功！')
               except:
                   try:
                       driver.find_element_by_xpath("//*[text()='当天已经提交过，是否继续提交？']")
                       print("已打卡过！")
                       driver.find_element_by_xpath("//a[@class='layui-layer-btn0']").click()
                   except:
                       print("打卡失败！请重新开始")
               driver.find_element_by_xpath("//a[@class='layui-layer-btn0']").click()
               time.sleep(3)
               print("即将退出程序...")
               driver.quit()
   
   # 主函数
   if __name__ == '__main__':
       print("这是一个自动健康打卡的程序。\n\n请在同目录下的config.json中配置打卡信息，例如：\n'学号': '1827405055',\n'密码': '12345678',\n'现人员位置': '在苏州',      (请注意只可填入以下四项中的任意一项：'留校', '在苏州', '江苏省内其他地区', '在其他地区')\n'家庭地址': '工业园区'\n其余所有属性均为打卡系统自动填充的上一次打卡信息\n\n程序即将启动...")
       driver = openChrome()
       operate_dk(driver)
   ```

   保存，然后在同目录下新建一个`config.json`文件用来存放个人信息，自行修改即可，注意人员位置只可以填写以下四项中的任意一项：‘留校’, ‘在苏州’, ‘江苏省内其他地区’, ‘在其他地区’

   ```json
   {
       "学号": "xxxxxx",
       "密码": "xxxxxx",
       "现人员位置": "江苏省内其他地区",
       "家庭地址": "home"
   }
   ```

   ## 运行

   最后python3 data.py即可自动打卡

   ![image-20200723074142728](https://imgconvert.csdnimg.cn/aHR0cHM6Ly90dmExLnNpbmFpbWcuY24vbGFyZ2UvMDA3UzhaSWxneTFnaDBrYnl6ZG1tajMxYTIwZDRuNGQuanBn?x-oss-process=image/format,png)

   可以通过定时任务来自动化进行打卡操作

   ## 脑洞大开

   可以在打卡成功后自定义推送给自己的邮箱或者qq微信来实现推送功能

   ## 致谢名单

   1. [CentOS7下无界面使用Selenium+chromedriver进行自动化测试](https://blog.csdn.net/pengjunlee/article/details/91997908?)

   2. [自动健康打卡程序](https://blog.csdn.net/weixin_44739303/article/details/106245788)

   3. [INSTALLING GOOGLE CHROME ON CENTOS, AMAZON LINUX, OR RHEL](https://intoli.com/blog/installing-google-chrome-on-centos/)

   4. [MAC安装chromedriver提示“Message: 'chromedriver' executable needs to be in PATH.”](https://blog.csdn.net/walter_chan/article/details/50464625)