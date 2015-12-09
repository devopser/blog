---
date: 2015-12-10 04:46:31.000Z
layout: post
title: mysql接口与云信加密验证
tags: openvpn 云信验证码 mysql
categories: openvpn mysql
---
```python
# coding:utf-8
import os,sys,MySQLdb,commands,time,pycurl,cStringIO;
from hashlib import sha1
from hmac import new as hmac
class checkvpn:
    #运行日志格式
    def log(self,content,f='login_err.log'):
        log = content + ' | ' + time.strftime('%Y-%m-%d %H:%M:%S')
        log = log.replace("'","")
        #print log
        commands.getstatusoutput('echo "' + log + '">>/etc/openvpn/bin/' + f)

    #连接数据库
    def conn_mysql(self):
        try:
           ip='192.168.6.229'
           user='openvpn'
           pwd='openvpn'
           db='openvpn'
           conn = MySQLdb.connect(host=ip,user=user,passwd=pwd,db=db,charset="utf8")
           return conn
        except MySQLdb.Error,e:
           self.log('rasp_srv.sqlhelper',e.args[1])

     #执行数据库命令
    def exec_mysql(self,sqlcmd):
        try:
           conn=self.conn_mysql()
           cur = conn.cursor()
           cur.execute(sqlcmd)
           #cur.execute('commit')
           fet = cur.fetchone()
           print fet
           result = ''
           if fet:
              result = fet[0]
           cur.close()
           conn.close()
           return result
        except MySQLdb.Error,e:
           print sqlcmd
           self.log('rasp_srv.sqlhelper',e.args[1] + sqlcmd)

    #清除登录日志
    def clear_log(self,user):
        self.log(user)
        self.exec_mysql("call checkvpn_clearlog('"+user+"')")

    #调用过程记录本次登录
    def record_log(self,user):
        return self.exec_mysql("call checkvpn_record_log('" + user + "')")

    #调用过程检查账号密码与邮件
    def check_mysql(self,user,pwd,email):
        sqlcmd="call openvpn.checkvpn('" + user + "','" + pwd + "','" + email + "')"
        return self.exec_mysql(sqlcmd)

    #获取是否需云信验证标识
    def getmark_phonepwd(self,user):
        sqlcmd="select mark from users where status='Y' and username='" + user + "'"
        result =  self.exec_mysql(sqlcmd)
        return result
    #检查云信口令
    def check_phonepwd(self,user,phonepwd):
        def curl(url):
            buf = cStringIO.StringIO()
            c = pycurl.Curl()
            c.setopt(c.URL, url)
            c.setopt(c.WRITEFUNCTION, buf.write)
            c.perform()
            result = buf.getvalue()
            buf.close()
            return result
        #####调用云信API，获取token
        secret_key='3B08D6E686458465B069463628C23F60ACC14EEC'
        secret_str="/oauth2/v2/token/access_token?response_type=token&client_id=059A626B4A2F9D8C72B0&refresh_token=&state=123456"
        sign = hmac(secret_key, secret_str, sha1).hexdigest()
        tokenurl="https://api-onlinevreelc.cloudentify.com" + secret_str + "&sign="+sign
        result = eval(curl(tokenurl))
        if result['code']!=0:
           checkvpn().log('获取token失败')
           return False
        token=result['access_token']
        #####调用云信API，较验用户+验证码
        secret_str='/oauth2/v2/phone/otpuserauth?access_token=' + token + \
                  '&userid='+ user +'&otp=' + phonepwd +'&phonesn=&state=123456'
        sign = hmac(secret_key, secret_str, sha1).hexdigest()
        verifyurl='https://api-onlinevreelc.cloudentify.com' + secret_str + '&sign=' + sign
        result=eval(curl(verifyurl))
        if result['code']==0:
           return True
        return False
        ```




 

