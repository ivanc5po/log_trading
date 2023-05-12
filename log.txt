from datetime import date
import os
import openai
import requests
import json
from lxml import html
import requests

import time

text = ""

def get_show():

    today = date.today()
    day = today.strftime("%y/%m/%d")
    y = "20"+day.split("/")[0]
    m = day.split("/")[1]
    d = day.split("/")[2]

    def month(x):
        return "jan feb mar apr may jun jul aug sep oct nov dec".split()[x-1]

    page = requests.get("https://blockcast.it/"+day+"/madman-column-"+y+"-"+month(int(m))+"-"+d+"/")
    tree = html.fromstring(page.content)
    text = ""

    for i in range(100):
        for j in range(84587, 84587+365):
            try:
                text += tree.xpath('//*[@id="post-'+str(j)+'"]/div/p['+str(i+1)+']/text()')[0]
            except:
                pass

    print(text)
    text=text+"\n根據這篇文章判斷作者在文章中的看法可能為看漲或是看跌還是震盪"


    # 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
    def analyze_res(data):
        # 將 bytes 轉換為 string
        data_str = data.decode('utf-8')

        # 解析 JSON 資料
        json_data = json.loads(data_str)

        # 取出 choices 中的回應內容
        response = json_data['choices'][0]['message']['content']
        return response

    openai.api_key = "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"

    url='https://api.openai.com/v1/chat/completions'

    # 設置對話開始的提示語
    prompt = ""

    a, b, c = 0, 0, 0
    print("狂人開始運作了...")
    for i in range(10):
        prompt=text
        payload={
            "model": "gpt-3.5-turbo",
            "messages": [{"role": "user", "content": prompt}]
            #"n": 10,
            #"temperature: 0.7
        }

        headers={
            "Authorization": f"Bearer {openai.api_key}",
            "Content-Type":"application/json"
        }

        try:
            r = requests.post(url, data=json.dumps(payload), headers=headers)

            #解析回傳資料 僅取出r.content中 json格式中的content
            res=analyze_res(r.content)

            prompt +=  res+'\n'

            if "震盪" in res:
                c += 1
            if "看跌" in res:
                b += 1
            if "看漲" in res:
                a += 1

        except:
            print("狂人發生錯誤:",r.content)

    os.system("clear")
    if a > b and a > c:
        return "20"+day+" 狂人看說會漲，可以空爛了"
    elif a < b and b > c:
        return "20"+day+" 狂人說看會跌，可以 all in 多單了"
    else:
        return "20"+day+" 狂人說行情方向未知，不下單..."
    
import asyncio
import websockets

T = time.time()
text = get_show()
async def echo(websocket, path):
    print('echo')
    async for message in websocket:
        print(message,'received from client')
        greeting = f"Hello {message}!"
        if time.time()-T > 300:
            text = get_show()
            T = time.time()
        await websocket.send(text)
        print(f"> {text}")

asyncio.get_event_loop().run_until_complete(websockets.serve(echo, '103.212.97.147', 65432))
asyncio.get_event_loop().run_forever()
