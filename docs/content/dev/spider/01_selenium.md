---
title: selenium
---

selenium的技巧和案列。

``` {.python}
import pickle
import os.path
import time
from selenium import webdriver
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support import expected_conditions as EC
# from selenium.webdriver.common.action_chains import ActionChains

driver = webdriver.Firefox()
# cookie = input("请输入cookie(JSESSIONID):")
baseurl = "http://flyhi.toprp.com"
url = "http://flyhi.top/project/91/story_catalog/issue_list"
driver.get(url)
cookies = ""

time.sleep(18) # 睡眠等待人工登录
# 加载保存的cookies
# if os.path.isfile("cookies.txt"):
#   with open("cookies.txt", "r") as fp:
#       cookies = json.load(fp)
#       for cookie in cookies:
#           cookie.pop('domain') # 如果报domain无效的错误
#           driver.add_cookie(cookie)
if os.path.isfile("cookies.pk11"):
    cookies = pickle.load(open("cookies.pk1", "rb"))
    for cookie in cookies:
        # cookie.pop('domain') # 如果报domain无效的错误
        driver.add_cookie(cookie)

else:
    # 使用帐号密码登录
    driver.get(url)
    username = driver.find_element_by_xpath("//div/input[@placeholder='请输入域账号']").send_keys("zm01")
    password = "test"
    sub = driver.find_element_by_tag_name("button")
    sub.click()
    # 保存cookies
    # cookies = driver.get_cookies()
    # with open("cookies.txt", "w") as fp:
    #   json.dump(cookies, fp)
    pickle.dump(driver.get_cookies(), open("cookies.pk1", "wb"))

driver.get(url)
# 定位到需要操作的每一列
# 设置每页显示的数量100条
wait = WebDriverWait(driver, 10)
showPageSet = wait.until(EC.element_to_be_clickable((By.XPATH, "//div[span[text()='条/每页']]/div")))
showPageSet.click()
setPageNum = driver.find_elements_by_xpath("//div[text()='50']")[0].click()

pageBar = driver.find_element_by_xpath("//div[span[text()='第']]")
pageBar.click()
pageDiv = driver.find_element_by_css_selector("div.q-popover.pjm-selector-popover.column.no-wrap.font-13.animate-popup-up")
pages = pageDiv.find_elements_by_class_name("ellipsis")
pageBar.click()

for p in pages:
    # 循环点击页码
    # tmpPageBar = driver.find_element_by_xpath("//div[span[text()='第']]")
    # tmpPageBar.click()
    # tmpPageDiv = driver.find_element_by_css_selector("div.q-popover.pjm-selector-popover.column.no-wrap.font-13.animate-popup-up")
    # tmpPages = tmpPageDiv.find_elements_by_class_name("ellipsis")
    # tmpPages[pages.index(p)].click()
    if pages.index(p) == 0:
        pass
    else:
        pageBar.click()
        p.click()


    trs = driver.find_elements_by_xpath("//tr[td[text()='zm']]")
    for tr in trs:
        # if "待开发" in tr.text:
        if "待开发" in tr.text :
            tr.find_elements_by_tag_name("td")[-2].click()

            wait.until(EC.element_to_be_clickable((By.XPATH, "//div[text()='开始研发']"))).click()
            # 选择弹出框div
            popDiv = wait.until(EC.element_to_be_clickable((By.XPATH, "//div[div/div/span[text()='变更状态：']]")))
            iTags = popDiv.find_elements_by_tag_name("i")
            # skip 0 index,its error
            iTags[1].click()
            driver.find_elements_by_css_selector(".text-primary.font-13.cursor-pointer.pp-selectable-bg")[0].click()
            iTags[2].click()
            driver.find_elements_by_css_selector(".text-primary.font-13.cursor-pointer.pp-selectable-bg")[0].click()
            # iTags[3].click()
            # driver.find_element_by_xpath("//input[@placeholder='通过关键字查找']").send_keys("zm01")
            # 输入时间
            popDiv.find_element_by_xpath("//td/div/input[@placeholder='待输入']").send_keys("6")
            # 选择测试负责人
            iTags[5].click()
            driver.find_element_by_xpath("//input[@placeholder='通过关键字查找']").send_keys("zm01")
            person = wait.until(EC.element_to_be_clickable((By.XPATH, "//div[i[text()='person']]")))
            person.click()
            popDiv.click() # 点击隐藏弹出框
            # popDiv.find_element_by_css_selector(".col-grow.bg-transparent").send_keys("6")
            popDiv.find_element_by_css_selector("button.q-btn.pp-radius-4.font-13.bg-ok-light.text-white.pp-selectable-opacity-8").click()

    trs = driver.find_elements_by_xpath("//tr[td[text()='zm']]")
    for tr in trs:
        if "开发中" in tr.text:
            tr.find_elements_by_tag_name("td")[-2].click()
            wait.until(EC.element_to_be_clickable((By.XPATH, "//div[text()='研发完成']"))).click()
            popDiv = wait.until(EC.element_to_be_clickable((By.XPATH, "//div[div/div/span[text()='变更状态：']]")))
            popDiv.find_element_by_css_selector(".col-grow.bg-transparent").send_keys("6")
            popDiv.find_element_by_css_selector("button.q-btn.pp-radius-4.font-13.bg-ok-light.text-white.pp-selectable-opacity-8").click()

    trs = driver.find_elements_by_xpath("//tr[td[text()='zm']]")
    for tr in trs:
        if "开发完成待测试" in tr.text:
            tr.find_elements_by_tag_name("td")[-2].click()
            wait.until(EC.element_to_be_clickable((By.XPATH, "//div[text()='测试中']"))).click()
            popDiv = wait.until(EC.element_to_be_clickable((By.XPATH, "//div[div/div/span[text()='变更状态：']]")))
            iTags = popDiv.find_elements_by_tag_name("i")
            iTags[1].click()
            driver.find_elements_by_css_selector(".text-primary.font-13.cursor-pointer.pp-selectable-bg")[0].click()
            # iTags[2].click()
            # driver.find_element_by_xpath("//input[@placeholder='通过关键字查找']").send_keys("zm01")
            # person = wait.until(EC.element_to_be_clickable((By.XPATH, "//div[i[text()='person']]")))
            # person.click()
            # popDiv.click() # 点击隐藏弹出框
            popDiv.find_element_by_css_selector(".col-grow.bg-transparent").send_keys("2")
            popDiv.find_element_by_css_selector("button.q-btn.pp-radius-4.font-13.bg-ok-light.text-white.pp-selectable-opacity-8").click()

    trs = driver.find_elements_by_xpath("//tr[td[text()='zm']]")
    for tr in trs:
        if "测试中" in tr.text:
            tr.find_elements_by_tag_name("td")[-2].click()
            wait.until(EC.element_to_be_clickable((By.XPATH, "//div[text()='测试完成']"))).click()
            popDiv = wait.until(EC.element_to_be_clickable((By.XPATH, "//div[div/div/span[text()='变更状态：']]")))
            popDiv.find_element_by_css_selector(".col-grow.bg-transparent").send_keys("2")
            popDiv.find_element_by_css_selector("button.q-btn.pp-radius-4.font-13.bg-ok-light.text-white.pp-selectable-opacity-8").click()
# 
# elem = driver.find_element_by_name("q")
# elem.clear()
# elem.send_keys("pycon")
# elem.send_keys(Keys.RETURN)
# assert "No results found." not in driver.page_source
driver.close()
```
