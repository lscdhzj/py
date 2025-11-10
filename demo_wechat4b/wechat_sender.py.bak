# only test wechat
# 安装所需库
# pip install PyYAML
# pip install pytest
# pip install requests

# DOC REF
# https://www.runoob.com/w3cnote/yaml-intro.html
# https://www.runoob.com/python3/python3-tutorial.html

import requests
import json
import yaml
import os
import sys
import datetime
import re
import calendar
from zoneinfo import ZoneInfo

def load_config(config_path: str = "config.yaml") -> dict:
    try:
        if not os.path.exists(config_path):
            raise FileNotFoundError(f"配置文件不存在: {config_path}")
        with open(config_path, 'r', encoding='utf-8') as file:
            config = yaml.safe_load(file)
        if config is None:
            return {}
        return config
    except yaml.YAMLError as e:
        raise yaml.YAMLError(f"YAML 解析错误: {e}")
    except Exception as e:
        raise Exception(f"加载配置失败: {e}")

def send_wechat_message(webhook_url, msg):
    msg_type = "markdown"
    if msg_type == "text":
        data = {
            "msgtype": "text",
            "text": {
                "content": msg
            }
        }
    elif msg_type == "markdown":
        data = {
            "msgtype": "markdown",
            "markdown": {
                "content": msg
            }
        }
    else:
        print(f"不支持的消息类型: {msg_type}")
        return False

    try:
        headers = {'Content-Type': 'application/json'}
        response = requests.post(webhook_url, headers=headers, data=json.dumps(data), timeout=10)
        response.raise_for_status()  # 如果状态码不是200-399，抛出HTTPError异常
        result = response.json()
        if result.get('errcode') == 0:
            print("消息发送成功")
            return True
        else:
            print(f"业务逻辑失败: errcode={result.get('errcode')}, errmsg={result.get('errmsg')}")
            return False
    except requests.exceptions.Timeout:
        print("请求超时")
        return False
    except requests.exceptions.ConnectionError:
        print("网络连接错误")
        return False
    except requests.exceptions.HTTPError as e:
        print(f"HTTP错误: {e}")
        return False
    except json.JSONDecodeError:
        print("响应JSON解析失败")
        return False
    except Exception as e:
        print(f"未知错误: {e}")
        return False

def process_daily_reading_msg(msg: str):
    """
    解析每日早读信息。如果当天是周末，不打印任何内容。
    如果是工作日，则过滤 msg 中所有的周六日行，并高亮当天内容。
    """
    today = datetime.date.today()
    current_year = today.year
    today_weekday = today.weekday()  # 0-周一 1-周二 2-周三 3-周四 4-周五 5-是周六 6-是周日

    # md 格式高亮
    WARNING_BOLD_START = '<font color="warning">**'
    END = '**</font>'

    # --- 新增的判断逻辑：如果当前日期是周六或周日，直接返回，不打印任何内容 ---
    if today_weekday >= 5:
        print(f"今天是 {today.strftime('%Y-%m-%d')} ({calendar.day_name[today_weekday]})，属于周末。根据要求，不打印任何内容。")
        return ""
    print(f"今天是 {today.strftime('%Y-%m-%d')} ({calendar.day_name[today_weekday]})，是工作日，正在处理内容...")

    output_lines = []
    header_lines = []
    data_lines = []

    lines = msg.strip().split('\n')
    is_data_section = False
    for line in lines:
        stripped_line = line.strip()
        if not stripped_line:
            continue
        if "日期 读书内容 页数" in stripped_line:
            is_data_section = True
            header_lines.append(stripped_line)
            continue
        if not is_data_section:
            header_lines.append(stripped_line)
        else:
            data_lines.append(stripped_line)
    # 打印头部
    # for line in header_lines:
    #     print(f"-----{line}-----")

    for data_line in data_lines:
        match = re.match(r'(\d+月\d+日)\s+(.*)\s+(\d+-\d+)', data_line)
        if match:
            date_str_with_char = match.group(1)
            print(f"match:{match}")
            # 构造日期对象
            try:
                month_day_str = date_str_with_char.replace('月', '-').replace('日', '')
                full_date_str = f"{current_year}-{month_day_str}"
                item_date = datetime.datetime.strptime(full_date_str, '%Y-%m-%d').date()
            except ValueError:
                continue
            # 过滤掉 msg 中周末的行
            if item_date.weekday() >= 5:
                continue
            # 检查是否为今天
            is_today = (item_date == today)
            # 构建输出行
            output_line = data_line
            # 高亮今天的行
            if is_today:
                output_line = f"{WARNING_BOLD_START}{output_line}{END}"
        output_lines.append(output_line)
        # print(output_line)
    output_lines = header_lines + output_lines
    print(output_lines)
    msg = '\n'.join(output_lines)
    return msg

def main():
    # 创建东八区时区
    east8 = ZoneInfo('Asia/Shanghai')

    # 获取当前东八区时间
    now_east8 = datetime.datetime.now(east8)
    print(f"当前东八区时间: {now_east8}")

    # 格式化输出
    formatted_time = now_east8.strftime("%Y-%m-%d %H:%M:%S %Z%z")
    print(f"格式化时间: {formatted_time}")

    # 注意XX月XX日格式日期后面要跟一个空格
    msg = f"""
        每日早读
        时间：{json.dumps(formatted_time, ensure_ascii=False)}
        日期 读书内容 页数
        11月10日 班上另外一名学员 温和友善，宽以待人 132-133
        11月11日 5.循循善诱 让他乐于接受我的建议 134-135
        11月12日 约瑟夫·艾利森是 成了这家店的常客 136-137
        11月13日 苏格拉底是世界上 最大的一笔订单 138-139
        11月14日 我知道，如果我没有失声 在课上分享时说， 140-141
        11月17日 终于有一天 以免招致猴妒与恨 142-143
        11月18日 提出建议进行引导 也该听听我的想法了， 144-146
        11月19日 到的 146-148
        11月20日 8.换位思考 方便的时候补上就可以了， 149-150
        11月21日 下次，当你想让别人 好好和她说说 151-153
        11月24日 但事实上很抱款写信斥责我 153-154
        11月25日 如果你遇到下面这样的困境 感同身受 155-156
        11月26日 让对方深信缠的人那里要到账 157-158
        11月27日 你的怀疑很合理诚实正直的好人 159-160
        11月28日 用戏剧化的方式，但很难对付 161-162
        """
    success = False
    msg = process_daily_reading_msg(msg)
    print(msg)
    if msg:
        webhook_url = os.getenv('WECHAT_WEBHOOK_URL') # 从环境变量或 secrets 获取 webhook URL
        if webhook_url:
            print("加载云端config")
        else:
            print("加载本地config")
            config = load_config()
            print(config.get('wechat_work', {}))
            webhook_url = config.get('wechat_work', {}).get('webhook_url')
        success = send_wechat_message(webhook_url, msg)
    sys.exit(0 if success else 1)

# 使用方法二的主函数
if __name__ == "__main__":
    main()