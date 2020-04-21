# Chrome的开发者模式
在“Network”面板勾选“Preserve log”复选框，然后进行登录，就可以记录HTTP POST信息，查看发送的表单信息详情。之后在贴吧首页开启开发者工具后再登录时，就可以看到下图所示的信息，其中“Form Data”就包含向服务器发送的表单信息详情。
![avatar](https://i.loli.net/2020/04/21/dh3jAurZO8MzKlW.png)

# 爬虫豆瓣电影TOP250
    from bs4 import BeautifulSoup
    import requests
    import os


    def get_html(web_url):
        header = {
            "User-Agent": "Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US) AppleWebKit/534.16 (KHTML, like Gecko) Chrome/10.0.648.133 Safari/534.16"}
        html = requests.get(url=web_url, headers=header).text
        Soup = BeautifulSoup(html, "lxml")
        data = Soup.find("ol").find_all("li")
        return data


    def get_info(all_move):
        f = open("douban.txt", "a")

        for info in all_move:
            #    排名
            nums = info.find('em')
            num = nums.get_text()

            pictures=info.find('img')
            picture=pictures.get('src')
            #    名字
            names = info.find("span")  
            name = names.get_text()

            #    导演
            charactors = info.find("p")  
            charactor = charactors.get_text().replace(" ", "").replace("\n", "")  # 使信息排列规律
            charactor = charactor.replace("\xa0", "").replace("\xee", "").replace("\xf6", "").replace("\u0161", "").replace(
                "\xf4", "").replace("\xfb", "").replace("\u2027", "").replace("\xe5", "")

            #    评语
            remarks = info.find_all("span", {"class": "inq"})
            if remarks:  # 这个判断是因为有的电影没有评语，你需要做判断
                remark = remarks[0].get_text().replace("\u22ef", "")
            else:
                remark = "此影片没有评价"
            print(remarks)

            # 评分
            scores = info.find_all("span", {"class": "rating_num"})
            score = scores[0].get_text()

            f.write(num + '、')
            f.write(name + "\n")
            f.write(picture + "\n")
            f.write(charactor + "\n")
            f.write(remark + "\n")
            f.write(score)
            f.write("\n\n")

        f.close()  # 记得关闭文件


    if __name__ == "__main__":

        if os.path.exists("douban.txt") == True:
            os.remove("douban.txt")

        page = 0  # 初始化页数，TOP一共有250部   每页25部
        while page <= 225:
            web_url = "https://movie.douban.com/top250?start=%s&filter=" % page
            all_move = get_html(web_url)  # 返回每一页的网页
            get_info(all_move)  # 匹配对应信息存入本地
            page += 25
