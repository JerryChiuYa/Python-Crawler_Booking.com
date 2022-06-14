# Web Crawler on Booking.com
## Purpose
<ul>
<li>When we reserves a hotel on some good holidays, there're so many fantastic rooms or facilities we want to live, but we have no time or moods (surely have money) to click the page seperately.</li>
<p></p>
<img src="/ImgForIntro/booking.png"/>

</ul>

#### Why not take the advantage of elegant Python codes to grab the information to an excel file?
<hr>


## Built With

<ul>
    <li><a href="https://jupyterlab.readthedocs.io/en/stable/">Jupyterlab 3.4.3</a></li>
    <li><a href="https://www.selenium.dev/">Selenium 4.2.0</a></li>
    <li><a href="https://chromedriver.chromium.org/downloads">ChromeDriver  102.0.5005.61</a></li>
</ul>
<hr>

## How to use
<ol>
<li>In the line 21, change the position of chromedriver.exe where you download.</li>
<li>In the line 22, change the booking.com where you want to travel.</li>
</ol>

```Python
browser=webdriver.Chrome(r"The position of chromedriver.exe", options=chrome_options)
browser.get("Your booking url")
```
<hr>

## OverView result
#### Here is the output:
<img src="/ImgForIntro/crawler_excel.png"/>
<hr>
