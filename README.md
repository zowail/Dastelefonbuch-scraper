# Welcome to dastelefonbuch.de scraper project

## The full source code is private since it's a paid software, contact me for more details

### In this page i will talk about some parts of the code including a sample from the source code.

**Starting from this url** : https://kontakt-1.dastelefonbuch.de/Rostock/L-500063366201-abass-fadil.html

The following code is written in **python 3.6.5** and **Selenium** which is a module for scraping and automating.

![Image of search results page](https://raw.githubusercontent.com/zowail/Dastelefonbuch-scraper/master/profile%20page.PNG)

**Basically the full script is searching for names in dastelefonbuch.de and then scrapes all names and phones from all pages in the search results**

**But in this page i will present only the part of scraping one search result page and not all of the website**

The script will open google chrome using the chrome driver then visits a search results page for any name e.g. "abass fadil"
and then will extract any contact details "Names and Phone Numbers" but if the name string has more than 2 words then it will skip it and also if there is no phone number it will be skipped as well.

**First import the needed modules**
```python
from time import sleep
from selenium.webdriver.common.keys import Keys
from selenium import webdriver
from parsel import Selector
import csv
import os.path
from collections import defaultdict
```

**parse name function**
```python
def parse_name_from_profile():
    sel = Selector(text=driver.page_source)
    try:
        # get profile name from h1 tag
        profile_name = sel.css("h1::text").extract_first()
        # remove spaces from the string
        profile_name = (str(profile_name)).strip()
    except:
        profile_name = "None"
    return profile_name
```
**check if a string has numbers or not**
```python
def hasNumbers(inputString):
    return any(char.isdigit() for char in inputString)
```

**count how many phone web elements in the parent web element to know how many phone numbers**
```python
def check_number_of_phone_containers():
    try:
        # find all containers with class named "nr"
        all_phone_containers = driver.find_elements_by_css_selector("span.nr")

        # check number of phone containers
        if( len(all_phone_containers) == 0):
            return 0

        elif(len(all_phone_containers) == 1):
            return 1
        # if the number is more than 1 return number 2 so it's a special case
        else:
            return 2
    except:
        return 0
```

**parse phone number**
```python
def parse_phone_number_from_profile():
    sel = Selector(text=driver.page_source)
    try:
        phone_number_html_strings = sel.css("span.nr *::text").extract()
    except:
        phone_number_html_strings = []
        
    # if the list is empty so there is no phone number
    if ( not phone_number_html_strings ):
        return "None"

    # if the profile contains a phone number
    else:
        
        # check first if the list contains any string with " " in it to handle this special case
        special_case_flag = 0

        # check if hide_class is found in span so this is a special case that needs to handle
        hide_class_text = sel.css("span.nr span.hide::text").extract()
        if (not hide_class_text):
            special_case_flag = 0
        else:
            special_case_flag = 1
        
        # if it's the special case
        if special_case_flag == 1:
            # string that will contain the full phone number
            string_number = ""

            # find only the strings that doesn't contain spaces or the special string "\xa0"
            for string in phone_number_html_strings:
                if (" " not in string) and ("\xa0" not in string):
                    string_number += string

            # remove any spaces from the string
            string_number = string_number.strip()
            return string_number

        # if the flag is zero so there is no special case to handle
        elif special_case_flag == 0:
            # convert all elements to string and remove any spaces
            for i in range(len(phone_number_html_strings)):
                phone_number_html_strings[i] = (str(phone_number_html_strings[i])).strip()

            # string that will contain the full phone number
            string_number = ""
            
            # check if the string contains numbers then append the string to string_number
            for string in phone_number_html_strings:
                if hasNumbers(string):
                    string_number += string
                    
            # remove any spaces from the string
            string_number = string_number.replace(" ", "")
            string_number = string_number.strip()
            return string_number
```

**parse name function**
```python
def parse_name_from_profile():
    sel = Selector(text=driver.page_source)
    try:
        # get profile name from h1 tag
        profile_name = sel.css("h1::text").extract_first()
        # remove spaces from the string
        profile_name = (str(profile_name)).strip()
    except:
        profile_name = "None"
    return profile_name
```

**parse phone numbers but this functions is for the profiles with more than 1 phone number**
```python
def parse_phone_number_from_profile_contains_more_than_one():
    try:
        # find only the first container with class named "nr"
        first_phone_container = driver.find_element_by_css_selector("span.nr")

        phone_number = first_phone_container.text
        # convert to string and remove spaces
        phone_number = str(phone_number)
        phone_number = phone_number.strip()
        phone_number = phone_number.replace(" ", "")

        return phone_number
    except:
        return "None"
```

**the main part where we use the functions created**
```python
driver = webdriver.Chrome('C:\chromedriver\chromedriver.exe')


driver.get("https://kontakt-1.dastelefonbuch.de/Rostock/L-500063366201-abass-fadil.html")


sleep(1)


name = parse_name_from_profile()

print(name)

number_of_phone_containers = check_number_of_phone_containers()


# check number of containers with the class named "nr" to determine which function will be used in parsing
if (number_of_phone_containers == 0):
    
    print("No phone")
elif(number_of_phone_containers == 1):
    phone_number = parse_phone_number_from_profile()
    print(phone_number)

else:
    phone_number = parse_phone_number_from_profile_contains_more_than_one()
    print(phone_number)
```

![Image of CSV Output for some profiles with the name "Fadil"](https://raw.githubusercontent.com/zowail/Dastelefonbuch-scraper/master/profiles%20contact.PNG)

