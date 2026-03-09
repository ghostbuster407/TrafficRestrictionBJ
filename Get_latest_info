import requests
from bs4 import BeautifulSoup
import re
from datetime import datetime, timedelta

def get_limit_rules():
    url = "http://bj.bendibao.com/news/2018725/251071.shtm"
    headers = {'User-Agent': 'Mozilla/5.0 (iPhone; CPU iPhone OS 17_0 like Mac OS X)'}
    
    try:
        response = requests.get(url, headers=headers)
        response.encoding = 'utf-8'
        content = response.text
        
        # 匹配规则：星期一至星期五限行机动车车牌尾号分别为：X和Y
        # 这个正则兼容了官网常见的文字描述格式
        patterns = re.findall(r"星期([一二三四五]).*?尾号分别为[：:\s]*(\d)[和及与](\d)", content)
        
        # 将汉字星期映射为索引
        week_map = {"一": 0, "二": 1, "三": 2, "四": 3, "五": 4}
        rules = {}
        for w, p1, p2 in patterns:
            rules[week_map[w]] = f"{p1}和{p2}"
            
        return rules
    except Exception as e:
        print(f"抓取失败: {e}")
        return None

def create_ics(rules):
    if not rules or len(rules) < 5:
        print("未获取到完整限行规则，跳过生成")
        return

    ics_lines = [
        "BEGIN:VCALENDAR",
        "VERSION:2.0",
        "X-WR-CALNAME:北京限行(官网同步)",
        "X-WR-TIMEZONE:Asia/Shanghai",
        "CALSCALE:GREGORIAN"
    ]
    
    # 生成未来 90 天的日程，确保覆盖整个季度
    today = datetime.now()
    for i in range(90):
        day = today + timedelta(days=i)
        weekday = day.weekday()
        
        if weekday in rules:
            date_str = day.strftime("%Y%m%d")
            event = [
                "BEGIN:VEVENT",
                f"UID:bj-limit-{date_str}@gov.cn",
                f"DTSTART;VALUE=DATE:{date_str}",
                f"SUMMARY:北京限行尾号：{rules[weekday]}",
                "DESCRIPTION:数据自动抓取自北京市人民政府官网",
                "END:VEVENT"
            ]
            ics_lines.extend(event)
            
    ics_lines.append("END:VCALENDAR")
    
    with open("limit.ics", "w", encoding="utf-8") as f:
        f.write("\n".join(ics_lines))

if __name__ == "__main__":
    rules = get_limit_rules()
    create_ics(rules)
    print("limit.ics 已生成成功")
