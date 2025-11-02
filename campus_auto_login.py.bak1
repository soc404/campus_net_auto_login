import re
from urllib import request
import requests
import time
import random
import os
import logging
import json
from typing import Dict, Any, Optional

class CampusNetworkAutoLogin:
    def __init__(self, config_file: str = 'config.json'):
        self.config = self.load_config(config_file)
        self.setup_logging()
        self.network_check_count = 0
        
    def load_config(self, config_file: str) -> Dict[str, Any]:
        """从JSON配置文件加载参数"""
        try:
            if not os.path.exists(config_file):
                self.create_sample_config(config_file)
                raise FileNotFoundError(f"配置文件 {config_file} 不存在，已创建示例配置文件，请修改后重新运行")
            
            with open(config_file, 'r', encoding='utf-8') as f:
                config = json.load(f)
            
            return config
            
        except json.JSONDecodeError as e:
            raise ValueError(f"配置文件格式错误: {e}")
        except Exception as e:
            raise RuntimeError(f"加载配置失败: {e}")
    
    def create_sample_config(self, config_file: str):
        """创建示例配置文件"""
        sample_config = {
            "network": {
                "post_url": "http://192.168.102.166/eportal/InterFace.do?method=login",
                "get_url": "http://192.168.102.166/eportal/success.jsp?userIndex=YOUR_USER_INDEX&keepaliveInterval=0",
                "timeout": 10,
                "check_interval": 60,
                "quick_check_interval": 10,
                "max_quick_checks": 6,
                "internet_test_urls": [
                    "http://www.baidu.com",
                    "http://www.qq.com",
                    "http://www.163.com"
                ]
            },
            "login": {
                "userId": "YOUR_USERNAME",
                "password": "YOUR_ENCRYPTED_PASSWORD",
                "queryString": "YOUR_QUERY_STRING",
                "passwordEncrypt": "true",
                "operatorPwd": "",
                "operatorUserId": "",
                "validcode": "",
                "service": ""
            },
            "headers": {
                "Cookie": "YOUR_COOKIE",
                "User-Agent": "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36"
            },
            "logging": {
                "log_file": "netconnect.log",
                "max_log_size_kb": 1024,
                "log_level": "INFO"
            }
        }
        
        with open(config_file, 'w', encoding='utf-8') as f:
            json.dump(sample_config, f, indent=2, ensure_ascii=False)
        
        print(f"已创建示例配置文件: {config_file}")
        print("请修改配置文件中的参数后重新运行程序")
    
    def setup_logging(self):
        """设置日志系统"""
        log_config = self.config['logging']
        
        # 清理过大的日志文件
        if os.path.exists(log_config['log_file']) and \
           os.path.getsize(log_config['log_file']) > log_config['max_log_size_kb'] * 1024:
            open(log_config['log_file'], 'w').close()
        
        logging.basicConfig(
            level=getattr(logging, log_config['log_level']),
            format='%(asctime)s - %(levelname)s - %(message)s',
            handlers=[
                logging.FileHandler(log_config['log_file'], encoding='utf-8'),
                logging.StreamHandler()
            ]
        )
        self.logger = logging.getLogger(__name__)
    
    def check_internet_connectivity(self) -> bool:
        """检查真正的互联网连通性（不只是校园网登录状态）"""
        test_urls = self.config['network'].get('internet_test_urls', [
            "http://www.baidu.com",
            "http://www.qq.com", 
            "http://www.163.com"
        ])
        
        for url in test_urls:
            try:
                response = requests.get(url, timeout=5)
                if response.status_code == 200:
                    self.logger.debug(f"网络连通性检查通过: {url}")
                    return True
            except Exception as e:
                self.logger.debug(f"无法访问 {url}: {e}")
                continue
        
        return False
    
    def check_login_status(self) -> bool:
        """检查校园网登录状态"""
        try:
            network_config = self.config['network']
            response = request.urlopen(
                network_config['get_url'], 
                timeout=network_config['timeout']
            )
            html = response.read().decode('gbk', errors='ignore')
            
            title_match = re.findall(r'<title>(.*?)</title>', html)
            if not title_match:
                self.logger.warning("无法获取页面标题")
                return False
                
            title = title_match[0]
            self.logger.debug(f"当前页面标题: {title}")
            
            return title == '登录成功'
            
        except Exception as e:
            self.logger.debug(f"检查登录状态失败: {e}")
            return False
    
    def login(self) -> bool:
        """执行登录操作"""
        try:
            login_data = self.config['login'].copy()
            headers = self.config['headers'].copy()
            
            # 动态计算Content-Length
            import urllib.parse
            encoded_data = urllib.parse.urlencode(login_data)
            headers["Content-Length"] = str(len(encoded_data))
            
            response = requests.post(
                self.config['network']['post_url'], 
                data=login_data, 
                headers=headers,
                timeout=self.config['network']['timeout']
            )
            
            response.encoding = 'utf-8'
            self.logger.info(f"登录请求状态码: {response.status_code}")
            
            if response.status_code == 200:
                time.sleep(2)  # 等待登录完成
                return self.check_login_status()
            
            return False
            
        except Exception as e:
            self.logger.error(f"登录失败: {e}")
            return False
    
    def validate_config(self) -> bool:
        """验证配置是否完整"""
        required_fields = {
            'network': ['post_url', 'get_url'],
            'login': ['userId', 'password', 'queryString'],
            'headers': ['Cookie', 'Referer']
        }
        
        for section, fields in required_fields.items():
            if section not in self.config:
                self.logger.error(f"配置文件中缺少 {section} 部分")
                return False
            
            for field in fields:
                if field not in self.config[section] or not self.config[section][field]:
                    self.logger.error(f"配置文件中 {section}.{field} 为空或缺失")
                    return False
        
        return True
    
    def run_smart(self):
        """智能运行模式：只在断网时尝试登录"""
        if not self.validate_config():
            self.logger.error("配置文件验证失败，请检查配置文件")
            return
        
        self.logger.info("智能联网脚本开始运行...")
        self.logger.info(f"正常检查间隔: {self.config['network']['check_interval']} 秒")
        self.logger.info(f"快速检查间隔: {self.config['network']['quick_check_interval']} 秒")
        
        while True:
            try:
                # 检查互联网连通性
                internet_ok = self.check_internet_connectivity()
                login_ok = self.check_login_status()
                
                if internet_ok and login_ok:
                    # 网络正常，重置计数器，长时间休眠
                    if self.network_check_count > 0:
                        self.logger.info("网络已恢复正常，切换到正常检查模式")
                        self.network_check_count = 0
                    
                    base_interval = self.config['network']['check_interval']
                    sleep_time = base_interval + random.uniform(0, 5)
                    self.logger.info(f"网络正常，休眠 {sleep_time:.2f} 秒")
                    time.sleep(sleep_time)
                    
                elif not internet_ok and login_ok:
                    # 已登录但无法上网，可能是网络问题
                    self.logger.warning("已登录校园网但无法访问互联网，可能是网络故障")
                    quick_interval = self.config['network']['quick_check_interval']
                    sleep_time = quick_interval + random.uniform(0, 2)
                    self.logger.info(f"等待 {sleep_time:.2f} 秒后重新检查")
                    time.sleep(sleep_time)
                    
                else:
                    # 未登录或网络断开，尝试登录
                    self.network_check_count += 1
                    max_quick_checks = self.config['network']['max_quick_checks']
                    
                    self.logger.info(f"网络断开，尝试登录 ({self.network_check_count}/{max_quick_checks})")
                    
                    if self.login():
                        self.logger.info("登录成功")
                        self.network_check_count = 0
                        # 登录成功后立即检查网络
                        continue
                    else:
                        self.logger.warning("登录失败")
                    
                    # 断网时使用快速检查间隔
                    if self.network_check_count < max_quick_checks:
                        quick_interval = self.config['network']['quick_check_interval']
                        sleep_time = quick_interval + random.uniform(0, 2)
                    else:
                        # 达到最大快速检查次数，切换到正常间隔
                        base_interval = self.config['network']['check_interval']
                        sleep_time = base_interval + random.uniform(0, 5)
                        self.logger.info(f"连续检查{max_quick_checks}次后切换到正常检查模式")
                    
                    self.logger.info(f"休眠 {sleep_time:.2f} 秒后重新检查")
                    time.sleep(sleep_time)
                
            except KeyboardInterrupt:
                self.logger.info("用户中断脚本执行")
                break
            except Exception as e:
                self.logger.error(f"运行过程中发生错误: {e}")
                time.sleep(30)

def main():
    """主函数"""
    try:
        auto_login = CampusNetworkAutoLogin('config.json')
        auto_login.run_smart()  # 使用智能模式
    except Exception as e:
        print(f"程序启动失败: {e}")
        print("请检查配置文件是否正确")

if __name__ == "__main__":
    main()