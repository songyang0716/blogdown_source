---
title: Scrape daily stock price and deliver to your email
date: '2017-09-29'
Author: Yang Song
thumbnailImagePosition: left
thumbnailImage: "img/stock_pic.jpg"
---

A tutorial on website scraping & email delivering using python
<!--more-->





### Background
 As I think back, I've always been involved with the investment market unintentionally. As a kid, my parents always talk with their financial advisors about their mutual funds, look for new investment opportunities in the market. They urged me to buy some bonds with my pocket money during college. However, at that time, I didn't really know what money really was and meant. After I graduated from school around two years ago, I started thinking about investment. I have a little extra money that I can roll the dice with. I already contribute to my 401k and also have the majority of my money in the mutual funds, which apparently are less risky than a stock. However, I want to try some stocks to help me understand money and market better. I picked a couple of stocks and purchased a few shares in each of them. However, I am tired of checking the stock information using my phone. So I am thinking about to retrieve the stock indices automatically from the internet and then send to my personal email. One could also set up a automater to run the script every hour.

### Code Review
 The coding involves two methods,the first method is called scrape, which uses python [BeautifulSoup](http://web.stanford.edu/~zlotnick/TextAsData/Web_Scraping_with_Beautiful_Soup.html) package to extract the stock indices for specific stocks that we care about. The second method is called email, which will send the information to the specific email addresses using the python [smtplib](https://docs.python.org/3/library/smtplib.html) package.

#### Python Code

{{< codeblock "scraper.py" "python" "http://underscorejs.org/#compact" "scraper.py" >}}


import urllib2
from bs4 import BeautifulSoup
from time import sleep
import smtplib
import getpass
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from collections import defaultdict

class stock_scraper:
    def __init__(self, stocks = ['SPX:IND','INDU:IND','CCMP:IND','SNAP:US','DLPH:US', 'HD:US'], sender_email = 'sender email', receiver_email = ['receiver email']) :
        """
        Stocks : Default contains five stocks
        All using stocks ticker symbol

        sender_email : From which email address do you want to send the file
        receiver_email : Which email address to send the file
        """
        self.stock_performance = defaultdict(list)
        self.scrape(stocks)
        self.email(sender_email, receiver_email)

    def scrape(self, stocks) :
        """
        stocks : list of stocks ticker symbol
        """
        for stock in stocks:
            quote_page = 'http://www.bloomberg.com/quote/' + stock
            page = urllib2.urlopen(quote_page)
            soup = BeautifulSoup(page, 'html.parser')

            name_box = soup.find('h1', attrs={'class': 'name'})
            name = name_box.text
            price_box = soup.find('div', attrs = {'class':'price'})
            price = price_box.text
            change_box = soup.find('div', attrs = {'class':'change-container'})
            percent_change = change_box.text.strip().split('   ')[1]
            percent_change = float(percent_change.split("%")[0])

            if not soup.find('div', attrs = {'class':'price-container up'}):
                percent_change *= -1

            if percent_change and price and name:
                self.stock_performance[name].append(price)
                self.stock_performance[name].append(percent_change)

            sleep(2)

        return

    def email(self, sender, receivers) :
        """
        from : sender's email address
        to: receivers' email addresses
        """
        msg = MIMEMultipart()
        msg['Subject'] = 'Daily Stock Performance'
        msg['From'] = sender
        msg['To'] = ",".join(receivers)
        msg.preamble = 'Daily Stock Performance'

        message = "Hello!\n"
        message += "\n"
        for stock_name, value in self.stock_performance.iteritems():
            if value[1] >= 0 :
                message += (stock_name + 'is doing good today! ' + 'The stock price increases ' + str(value[1]) + '%, The current stock price is $' + str(value[0])) + '\n'
                message += "\n"
            else:
                message += (stock_name + 'is not doing good today! ' + 'The stock price decreases ' + str(value[1]) + '%, The current stock price is $' + str(value[0])) + '\n'
                message += "\n"


        msg.attach(MIMEText(message, 'plain'))

        pw = 'password'
        s = smtplib.SMTP(host='smtp.gmail.com', port=587)
        s.starttls()
        try:
            s.login(sender, pw)
            print "Your email is sent, thank you !"
        except SMTPAuthenticationError:
            print "Wrong password dude !"
            return

        s.sendmail(sender, receivers, msg.as_string())

        del msg
        s.quit()

        return

{{< /codeblock >}}