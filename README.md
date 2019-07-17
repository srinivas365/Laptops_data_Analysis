
# Web scraping - hp Laptops 

![alt text](laptop_EN.jpg)

**Web Scraping means extracting information from websites by parsing the HTML of the webpage.**

website used for scraping : https://store.hp.com/in-en/default/laptops-tablets.html

Libraries used:

- **requests** <br>
  Requests gets the web page for you

- **BeautifulSoup**<br>
  you need to parse the HTML from the page to retrieve the data. That is done by BeautifulSoup.

- **re**<br>
  regular expressions module used to make manipulations on data we got.

- **pandas**<br>
  To create the dataframe from the data we got by scraping and creating
  

  
### Importing the above modules


```python
import requests
from bs4 import BeautifulSoup
import re
import pandas as pd
```

### Let's get started

Each Laptop product is rendered in the webpage as follow

![alt text](lappyt.jpeg)


```python
prod_list=[]
def getData(products):
    '''
    Takes a List of products and scrapes the required specifications of each product and
    append them to the prod_list defined above.
    '''
    
    for product in products:
        
        #name 
        name=product.find('a',{"class":"product-item-link"}).text.strip()

        #rating of product is in div tag with attribute data-by-average-overall-rating
        rating=product.find('div',attrs={'data-bv-average-overall-rating' : True})['data-bv-average-overall-rating']

        #processor family
        processor=product.find('li',{'class':'processorfamily'})
        if processor is not None:
            processor=processor.text.strip()
        else:
            processor=None
            

        # processor name, generation, type
        if processor is not None:
            try:
                if processor.find('Intel')!=-1:
                    proc_company='Intel'
                    if processor.find('Core')!=-1:  # processor type and generation for Intel core processors
                        generation=re.findall(r'\d',processor)[0] 
                        proc_type=re.findall(r'i\d',processor)[0]
                    else:
                        temp=processor.split()
                        proc_type=' '.join(temp[1:-1]) # processor type for Intel pentium and other series processors

                elif processor.find('AMD')!=-1: # processor type for AMD processors
                    proc_company='AMD' 
                    temp=processor.split()
                    proc_type=temp[1]+' '+temp[2]
                    generation=None
                else:                     # processor type for other processors
                    proc_company=None
                    proc_type=None
                    generation=None
            except:
                print('something went wrong')
        else:
            proc_company=None
            proc_type=None
            generation=None
            

        # Os installed
        os_installed=product.find('li',{'class':'osinstalled'})
        if os_installed is not None:
            os_installed=os_installed.text
        else:
            os_installed=None
            

        # RAM
        ram=product.find('li',{'class':'memstdes_01'})
        if ram is not None:
            ram=ram.text
        else:
            ram=None

        #hard disk
        hd=product.find('li',{'class':'hd_01des'})
        if hd is not None:
            hd=hd.text
        else:
            hd=None

        
        # graphic card information is stored in <li> element with two different classes so multiple classes are used below
        if product.find('li',{'class':['graphicseg_01card_01','graphicseg_02card_01']}) is not None:
            graphic_card=product.find('li',{'class':['graphicseg_01card_01','graphicseg_02card_01']}).text
        else:
            graphic_card=None
            

        #display-type
        display_type=product.find('li',{'class':['display-displaydes']})
        if display_type is None:
            display_type=None
        else:
            display_type=display_type.text


        #price
        
        # actual price, discount and final price all stored in span tag with class price hence we get a list
        prices=product.find_all('span',{'class':'price'})    
        if len(prices)==3:
            aprice=prices[0].text[1:]
            fprice=prices[1].text[1:]
            dprice=prices[2].text[1:]
        else:
            aprice=fprice=prices[0].text[1:] 
            dprice='0'

        #included items
        inc_items=product.find_all('ul',{'class':'included'})

        items_list=[]

        if inc_items is not None:
            for i in inc_items:
                items_list=[item.text for item in i.find_all('li')]
                
        inc_items=','.join(items_list) # converting the list of included items into string seperated by commas

        print('Name:',name)
        print('rating:',rating)
        print("processor:",processor)
        print("processor_company:",proc_company)
        print("processor_type:",proc_type)
        print("generation:",generation)
        print("os_installed:",os_installed)
        print("ram:",ram)
        print("hard_disk:",hd)
        print("graphic_card:",graphic_card)
        print("display:",display_type)
        print("Actual_price:",aprice)
        print("final_price:",fprice)
        print("Discount:",dprice)
        print("included_items:",inc_items)
        
        
        # creating a dictionary with keys as specification names and values as their respective information
        prod_data={
            'Name':name,
            'rating':rating,
            'processor':processor,
            'processor_company':proc_company,
            'processor_type':proc_type,
            'generation':generation,
            'os_installed':os_installed,
            'ram':ram,
            'hard_disk':hd,
            'graphic_card':graphic_card,
            'display':display_type,
            'actual_price':aprice,
            'final_price':fprice,
            'discout':dprice,
            'included_items':inc_items
        }
        
        #Appending the each product to list
        prod_list.append(prod_data)
        
        # end of the product
        print('-'*50)
        
    # end of all products
    print('*'*50)
```

All the products in website is rendered across 6 pages so we set a loop and get the all products of each page at time
and call the getData function defined above


```python
for page_count in range(1,7):
    url='https://store.hp.com/in-en/default/laptops-tablets.html?p='+str(page_count)+'&product_list_limit=30'
    data=requests.get(url) 
    data_soup=BeautifulSoup(data.text,'lxml')
    products=data_soup.find_all("div", {"class": "product-item-details"})
    getData(products)
```

    Name: HP ENVY x360 - 13-ag0035au
    rating: 3.6667
    processor: AMD Ryzen™ 5 processor
    processor_company: AMD
    processor_type: Ryzen™ 5
    generation: None
    os_installed: Windows 10 Home Single Language 64
    ram: 8 GB DDR4-2400 SDRAM (onboard)
    hard_disk: 256 GB SSD
    graphic_card: AMD Radeon™ Vega 8 Graphics
    display: 13.3" FHD multitouch-enabled edge-to-edge glass (1920 x 1080)
    Actual_price: 83,496
    final_price: 72,990
    Discount: 10,506
    included_items: 
    --------------------------------------------------
    Name: HP Gaming Pavilion - 15-cx0140tx
    rating: 5.0000
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Home Single Language 64
    ram: 8 GB DDR4-2666 SDRAM (1 x 8 GB)
    hard_disk: 1 TB 7200 rpm SATA
    graphic_card: NVIDIA® GeForce® GTX 1050 (4 GB GDDR5 dedicated)
    display: None
    Actual_price: 86,476
    final_price: 72,990
    Discount: 13,486
    included_items: HP Odyssey backpack (Worth ₹3,499),Microsoft Office Home and Student
    --------------------------------------------------
    Name: HP Notebook - 15-da0435tx
    rating: 4.0000
    processor: 7th Generation Intel® Core™ i3 processor
    processor_company: Intel
    processor_type: i3
    generation: 7
    os_installed: Windows 10 Home Single Language 64
    ram: 8 GB DDR4-2133 SDRAM (1 x 8 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: NVIDIA® GeForce® MX110 (2 GB DDR3 dedicated)
    display: None
    Actual_price: 50,292
    final_price: 44,580
    Discount: 5,712
    included_items: 
    --------------------------------------------------
    Name: HP Notebook - 15g-dr0006tx
    rating: 4.0625
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Home Single Language 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: NVIDIA® GeForce® MX110 (2 GB DDR3 dedicated)
    display: None
    Actual_price: 66,137
    final_price: 58,991
    Discount: 7,146
    included_items: HP Original Laptop Bag (Worth ₹1,123),1 Year On-site Warranty,Microsoft Office Home and Student
    --------------------------------------------------
    Name: HP Notebook 15-da1030tu
    rating: 2.0000
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Home 64
    ram: 4 GB DDR4-2400 SDRAM
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: None
    display: None
    Actual_price: 50,720
    final_price: 46,990
    Discount: 3,730
    included_items: HP Original Laptop Bag (Worth ₹1,123),Microsoft Office Home and Student
    --------------------------------------------------
    Name: HP Pavilion - 15-cw0027au
    rating: 3.0000
    processor: AMD Ryzen™ 5 processor
    processor_company: AMD
    processor_type: Ryzen™ 5
    generation: None
    os_installed: Windows 10 Home Single Language 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 1 TB 5400 rpm SATA + 128 GB SSD
    graphic_card: AMD Radeon™ Vega 8 Graphics
    display: None
    Actual_price: 70,025
    final_price: 59,990
    Discount: 10,035
    included_items: HP Trendsetter Bag (Worth ₹2,356)
    --------------------------------------------------
    Name: HP Pavilion 13-an0045tu
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Home Single Language 64
    ram: 8 GB DDR4-2400 SDRAM (onboard)
    hard_disk: 128 GB M.2 SSD
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 75,379
    final_price: 72,876
    Discount: 2,503
    included_items: HP Original Laptop Bag (Worth ₹1,123)
    --------------------------------------------------
    Name: HP Pavilion 13-an0046tu
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Home Single Language 64
    ram: 8 GB DDR4-2400 SDRAM (onboard)
    hard_disk: 256 GB PCIe® NVMe™ M.2 SSD
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 75,875
    final_price: 75,875
    Discount: 0
    included_items: HP Original Laptop Bag (Worth ₹1,123)
    --------------------------------------------------
    Name: HP Pavilion 14-ce1000tu
    rating: 3.0000
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Home Single Language 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 256 GB SSD
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 69,490
    final_price: 59,990
    Discount: 9,500
    included_items: HP Trendsetter Bag (Worth ₹2,356),Microsoft Office Home and Student
    --------------------------------------------------
    Name: HP Pavilion 14-ce1000tx
    rating: 4.5000
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Home Single Language 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 256 GB PCIe® NVMe™ M.2 SSD
    graphic_card: NVIDIA® GeForce® MX150 (2 GB GDDR5 dedicated)
    display: 14" diagonal FHD IPS BrightView WLED-backlit (1920 x 1080)
    Actual_price: 78,590
    final_price: 71,990
    Discount: 6,600
    included_items: HP Trendsetter Bag (Worth ₹2,356),Microsoft Office Home and Student
    --------------------------------------------------
    Name: HP Pavilion 14-ce1001tx
    rating: 4.2857
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Home Single Language 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 1 TB 5400 rpm SATA + 128 GB SSD
    graphic_card: NVIDIA® GeForce® MX150 (2 GB GDDR5 dedicated)
    display: None
    Actual_price: 80,946
    final_price: 69,989
    Discount: 10,957
    included_items: HP Trendsetter Bag (Worth ₹2,356),Microsoft Office Home and Student
    --------------------------------------------------
    Name: HP Pavilion 14-ce1003tx
    rating: 4.0000
    processor: 8th Generation Intel® Core™ i7 processor
    processor_company: Intel
    processor_type: i7
    generation: 8
    os_installed: Windows 10 Home Single Language 64
    ram: 16 GB DDR4-2400 SDRAM (1 x 16 GB)
    hard_disk: 512 GB SSD
    graphic_card: NVIDIA® GeForce® MX150 (2 GB GDDR5 dedicated)
    display: None
    Actual_price: 108,460
    final_price: 103,775
    Discount: 4,685
    included_items: HP Trendsetter Bag (Worth ₹2,356),Microsoft Office Home and Student
    --------------------------------------------------
    Name: HP Pavilion 15-cs1000tx
    rating: 2.1071
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Home Single Language 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: NVIDIA® GeForce® MX130 (2 GB GDDR5 dedicated)
    display: None
    Actual_price: 70,668
    final_price: 62,990
    Discount: 7,678
    included_items: HP Trendsetter Bag (Worth ₹2,356),1 Year On-site Warranty,Microsoft Office Home and Student
    --------------------------------------------------
    Name: HP Pavilion 15-cs1052tx
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i7 processor
    processor_company: Intel
    processor_type: i7
    generation: 8
    os_installed: Windows 10 Home Single Language 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 2 TB 5400 rpm SATA
    graphic_card: NVIDIA® GeForce® MX150 (4 GB GDDR5 dedicated)
    display: None
    Actual_price: 88,547
    final_price: 81,990
    Discount: 6,557
    included_items: HP Original Laptop Bag (Worth ₹1,123),Microsoft Office Home and Student
    --------------------------------------------------
    Name: HP Pavilion x360 - 11-ad106tu
    rating: 4.0000
    processor: 8th Generation Intel® Core™ i3 processor
    processor_company: Intel
    processor_type: i3
    generation: 8
    os_installed: Windows 10 Home Single Language 64
    ram: 4 GB DDR4-2400 SDRAM (1 x 4 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 50,005
    final_price: 45,580
    Discount: 4,425
    included_items: HP Active Pen (Worth ₹3,145),HP Trendsetter Bag (Worth ₹2,356)
    --------------------------------------------------
    Name: HP Pavilion x360 - 14-cd0050tx
    rating: 4.2500
    processor: 8th Generation Intel® Core™ i3 processor
    processor_company: Intel
    processor_type: i3
    generation: 8
    os_installed: Windows 10 Home Single Language 64
    ram: 4 GB DDR4-2400 SDRAM (1 x 4 GB), 2 SODIMM slots (upgradeable to 12 GB)
    hard_disk: 1 TB 5400 rpm SATA SSHD
    graphic_card: 2 GB NVIDIA GeForce MX 130 Graphics
    display: None
    Actual_price: 64,887
    final_price: 60,580
    Discount: 4,307
    included_items: HP Original Laptop Bag (Worth ₹1,123),HP Active Pen (Worth ₹3,145),Microsoft Office Home and Student
    --------------------------------------------------
    Name: HP Pavilion x360 - 14-cd0053tx
    rating: 4.3333
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Home Single Language 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB), 2 SODIMM Slots (upgradeable to 12 GB)
    hard_disk: 1TB 5400 RPM SATA+ 16 GB SSD Intel Optan
    graphic_card: 2 GB NVIDIA GeForce MX 130 Graphics
    display: None
    Actual_price: 86,834
    final_price: 80,875
    Discount: 5,959
    included_items: HP Original Laptop Bag (Worth ₹1,123),HP Active Pen (Worth ₹3,145),Microsoft Office Home and Student
    --------------------------------------------------
    Name: HP Pavilion x360 - 14-cd0078tu
    rating: 3.1429
    processor: 8th Generation Intel® Core™ i3 processor
    processor_company: Intel
    processor_type: i3
    generation: 8
    os_installed: Windows 10 Home Single Language 64
    ram: 4 GB DDR4-2400 SDRAM (1 x 4 GB)
    hard_disk: 256 GB SSD
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 61,247
    final_price: 53,580
    Discount: 7,667
    included_items: HP Original Laptop Bag (Worth ₹1,123),HP Active Pen (Worth ₹3,145),Microsoft Office Home and Student
    --------------------------------------------------
    Name: HP Pavilion x360 - 14-cd0080tu
    rating: 2.0000
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Home Single Language 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB), 2 SODIMM Slots (upgradeable to 12 GB)
    hard_disk: 1 TB 5400 rpm SATA SSHD (8 GB NAND)
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 77,841
    final_price: 73,875
    Discount: 3,966
    included_items: HP Original Laptop Bag (Worth ₹1,123),HP Active Pen (Worth ₹3,145),Microsoft Office Home and Student
    --------------------------------------------------
    Name: HP Pavilion x360 - 14-cd0081tu
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Home Single Language 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 256 GB SSD
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 82,659
    final_price: 70,876
    Discount: 11,783
    included_items: HP Original Laptop Bag (Worth ₹1,123),HP Active Pen (Worth ₹3,145),Microsoft Office Home and Student
    --------------------------------------------------
    Name: HP ProBook 440 G6 Notebook PC
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 79,347
    final_price: 76,999
    Discount: 2,348
    included_items: HP Overnighter Backpack (Worth ₹2,499)
    --------------------------------------------------
    Name: HP Notebook - 14s-cf0055tu
    rating: 3.3333
    processor: 7th Generation Intel® Core™ i3 processor
    processor_company: Intel
    processor_type: i3
    generation: 7
    os_installed: Windows 10 Home 64
    ram: 4 GB DDR4-2133 SDRAM (1 x 4 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: Intel® HD Graphics 620
    display: 14" diagonal FHD IPS BrightView micro-edge WLED-backlit (1920 x 1080)
    Actual_price: 39,000
    final_price: 38,080
    Discount: 920
    included_items: HP Original Laptop Bag (Worth ₹1,123),Microsoft Office Home and Student
    --------------------------------------------------
    Name: HP Notebook - 15-db0209au
    rating: 5.0000
    processor: AMD Dual-Core A-Series processor
    processor_company: AMD
    processor_type: Dual-Core A-Series
    generation: None
    os_installed: Windows 10 Home Single Language 64
    ram: 4 GB DDR4-1866 SDRAM (1 x 4 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: AMD Radeon™ R3 Graphics
    display: 15.6" diagonal HD SVA BrightView micro-edge WLED-backlit (1366 x 768)
    Actual_price: 26,000
    final_price: 24,850
    Discount: 1,150
    included_items: HP Original Laptop Bag (Worth ₹1,123),1 Year On-site Warranty
    --------------------------------------------------
    Name: HP ProBook x360 440 G1 Notebook PC
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 256 GB SSD
    graphic_card: Intel® UHD Graphics 620
    display: 14" FHD Touchscreen Display
    Actual_price: 89,587
    final_price: 86,935
    Discount: 2,652
    included_items: HP Active Pen (Worth ₹3,145),HP Overnighter Backpack (Worth ₹2,499)
    --------------------------------------------------
    Name: HP EliteBook x360 1030 G3 Notebook PC
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i7 processor
    processor_company: Intel
    processor_type: i7
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2133 SDRAM (onboard)
    hard_disk: 512 GB SSD
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 176,627
    final_price: 175,084
    Discount: 1,543
    included_items: HP Active Pen (Worth ₹3,145),HP Original Bag  (Worth ₹1,499) (#5DD44PA)
    --------------------------------------------------
    Name: HP Notebook - 15-db0186au
    rating: 3.6667
    processor: AMD Ryzen™ 3 processor
    processor_company: AMD
    processor_type: Ryzen™ 3
    generation: None
    os_installed: Windows 10 Home Single Language 64
    ram: 4 GB DDR4-2400 SDRAM (1 x 4 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: AMD Radeon™ Vega 3 Graphics
    display: None
    Actual_price: 38,622
    final_price: 33,080
    Discount: 5,542
    included_items: HP Original Laptop Bag (Worth ₹1,123),Microsoft Office Home and Student
    --------------------------------------------------
    Name: HP ProBook 440 G5 Notebook PC
    rating: 0.0000
    processor: 7th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 7
    os_installed: Windows 10 Pro 64
    ram: 4 GB DDR4-2400 SDRAM (1 x 4 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: Intel® HD Graphics 620
    display: None
    Actual_price: 77,427
    final_price: 72,402
    Discount: 5,025
    included_items: HP Overnighter Backpack (Worth ₹2,499)
    --------------------------------------------------
    Name: HP Notebook - 15-da0327tu
    rating: 2.3636
    processor: 7th Generation Intel® Core™ i3 processor
    processor_company: Intel
    processor_type: i3
    generation: 7
    os_installed: Windows 10 Home Single Language 64
    ram: 4 GB DDR4-2133 SDRAM (1 x 4 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: Intel® HD Graphics 620
    display: None
    Actual_price: 36,481
    final_price: 34,830
    Discount: 1,651
    included_items: HP Original Laptop Bag (Worth ₹1,123),Microsoft Office Home and Student
    --------------------------------------------------
    Name: HP Notebook - 15-da0295tu
    rating: 2.0000
    processor: Intel® Pentium® Silver processor
    processor_company: Intel
    processor_type: Pentium® Silver
    generation: 7
    os_installed: Windows 10 Home Single Language 64
    ram: 4 GB DDR4-2400 SDRAM (1 x 4 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: Intel® UHD Graphics 605
    display: None
    Actual_price: 27,381
    final_price: 26,112
    Discount: 1,269
    included_items: HP Original Laptop Bag (Worth ₹1,123)
    --------------------------------------------------
    Name: HP Notebook - 15-da0326tu
    rating: 3.0000
    processor: 7th Generation Intel® Core™ i3 processor
    processor_company: Intel
    processor_type: i3
    generation: 7
    os_installed: Windows 10 Home Single Language 64
    ram: 4 GB DDR4-2133 SDRAM (1 x 4 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: Intel® HD Graphics 620
    display: None
    Actual_price: 35,962
    final_price: 33,830
    Discount: 2,132
    included_items: HP Original Laptop Bag (Worth ₹1,123)
    --------------------------------------------------
    **************************************************
    Name: HP EliteBook 830 G5 Notebook PC
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 512 GB SSD
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 122,227
    final_price: 121,159
    Discount: 1,068
    included_items: HP Business Backpack (Worth ₹4,200)
    --------------------------------------------------
    Name: HP EliteBook 840r G4 Notebook PC
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 1 TB 7200 rpm SATA+128 GB SSD
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 112,883
    final_price: 111,897
    Discount: 986
    included_items: HP Business Backpack (Worth ₹4,200)
    --------------------------------------------------
    Name: HP EliteBook 840r G4 Notebook PC
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i7 processor
    processor_company: Intel
    processor_type: i7
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 1 TB 7200 rpm SATA
    graphic_card: Intel® UHD graphics 620
    display: None
    Actual_price: 115,571
    final_price: 114,561
    Discount: 1,010
    included_items: HP Business Backpack (Worth ₹4,200)
    --------------------------------------------------
    Name: HP ProBook x360 440 G1 Notebook PC
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i3 processor
    processor_company: Intel
    processor_type: i3
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 4 GB DDR4-2400 SDRAM (1 x 4 GB)
    hard_disk: 256 GB SSD
    graphic_card: Intel® UHD Graphics 620
    display: 14" FHD Touchscreen
    Actual_price: 80,627
    final_price: 78,240
    Discount: 2,387
    included_items: HP Overnighter Backpack (Worth ₹2,499)
    --------------------------------------------------
    Name: HP EliteBook 840r G4 Notebook PC
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i7 processor
    processor_company: Intel
    processor_type: i7
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 16 GB DDR4-2400 SDRAM (1 x 16 GB)
    hard_disk: 1 TB 7200 rpm SATA+128 GB SSD
    graphic_card: Intel® UHD graphics 620
    display: None
    Actual_price: 127,987
    final_price: 126,869
    Discount: 1,118
    included_items: HP Business Backpack (Worth ₹4,200)
    --------------------------------------------------
    Name: HP EliteBook 840r G4 Notebook PC
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 1 TB 7200 rpm SATA
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 102,643
    final_price: 101,747
    Discount: 896
    included_items: HP Business Backpack (Worth ₹4,200)
    --------------------------------------------------
    Name: HP ProBook x360 440 G1 Notebook PC
    rating: 2.0000
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 512 GB SSD
    graphic_card: Intel® UHD Graphics 620
    display: 14" FHD Touchscreen
    Actual_price: 102,643
    final_price: 99,605
    Discount: 3,038
    included_items: HP Active Pen (Worth ₹3,145),HP Overnighter Backpack (Worth ₹2,499)
    --------------------------------------------------
    Name: HP Pavilion - 15-bc407tx
    rating: 5.0000
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Home Single Language 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 1 TB 5400 rpm SATA + 128 GB SSD
    graphic_card: NVIDIA® GeForce® GTX 1050 (4 GB GDDR5 dedicated)
    display: None
    Actual_price: 89,474
    final_price: 74,941
    Discount: 14,533
    included_items: HP Trendsetter Bag (Worth ₹2,356),1 Year On-site Warranty
    --------------------------------------------------
    Name: HP Pavilion - 15-bc406tx
    rating: 3.1667
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Home Single Language 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: NVIDIA® GeForce® GTX 1050 (4 GB GDDR5 dedicated)
    display: None
    Actual_price: 80,481
    final_price: 69,730
    Discount: 10,751
    included_items: HP Trendsetter Bag (Worth ₹2,356),1 Year On-site Warranty
    --------------------------------------------------
    Name: HP Pavilion x360 - 11-ad105tu
    rating: 2.0000
    processor: Intel® Pentium® processor
    processor_company: Intel
    processor_type: Pentium®
    generation: 8
    os_installed: Windows 10 Home Single Language 64
    ram: 4 GB DDR4-2400 SDRAM (1 x 4 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: Intel® UHD Graphics 605
    display: None
    Actual_price: 39,942
    final_price: 37,581
    Discount: 2,361
    included_items: HP Trendsetter Bag (Worth ₹2,356)
    --------------------------------------------------
    Name: HP ZBook 15v G5 Mobile Workstation
    rating: 
    processor: 16 GB DDR4-2666 SDRAM (1 x 16 GB)
    processor_company: None
    processor_type: None
    generation: None
    os_installed: Windows 10 Pro 64
    ram: None
    hard_disk: 1 TB 7200 rpm SATA
    graphic_card: NVIDIA® Quadro® P600 (4 GB GDDR5 dedicated)
    display: None
    Actual_price: 225,937
    final_price: 203,842
    Discount: 22,095
    included_items: HP Original Bag  (Worth ₹1,499) (#5DD44PA),3 Years Warranty
    --------------------------------------------------
    Name: HP ZBook 15v G5 Mobile Workstation
    rating: 
    processor: None
    processor_company: None
    processor_type: None
    generation: None
    os_installed: Windows 10 Pro 64
    ram: 16 GB DDR4-2666 SDRAM (1 x 16 GB)
    hard_disk: 256 GB SSD
    graphic_card: NVIDIA® Quadro® P600 (4 GB GDDR5 dedicated)
    display: None
    Actual_price: 233,611
    final_price: 210,750
    Discount: 22,861
    included_items: HP Original Bag  (Worth ₹1,499) (#5DD44PA),3 Years Warranty
    --------------------------------------------------
    Name: HP ZBook 15v G5 Mobile Workstation
    rating: 
    processor: None
    processor_company: None
    processor_type: None
    generation: None
    os_installed: Windows 10 Pro 64
    ram: 16 GB DDR4-2666 SDRAM (1 x 16 GB)
    hard_disk: 1 TB 7200 rpm SATA
    graphic_card: NVIDIA® Quadro® P600 (4 GB GDDR5 dedicated)
    display: None
    Actual_price: 216,245
    final_price: 195,120
    Discount: 21,125
    included_items: HP Original Bag  (Worth ₹1,499) (#5DD44PA),3 Years Warranty
    --------------------------------------------------
    Name: HP ZBook 15v G5 Mobile Workstation
    rating: 
    processor: None
    processor_company: None
    processor_type: None
    generation: None
    os_installed: Windows 10 Pro 64
    ram: 16 GB DDR4-2666 SDRAM (1 x 16 GB)
    hard_disk: 2 TB 5400 rpm SATA
    graphic_card: NVIDIA® Quadro® P600 (4 GB GDDR5 dedicated)
    display: None
    Actual_price: 203,281
    final_price: 183,453
    Discount: 19,828
    included_items: HP Original Bag  (Worth ₹1,499) (#5DD44PA),3 Years Warranty
    --------------------------------------------------
    Name: HP ZBook 15v G5 Mobile Workstation
    rating: 
    processor: None
    processor_company: None
    processor_type: None
    generation: None
    os_installed: Windows 10 Pro 64
    ram: 16 GB DDR4-2666 SDRAM (1 x 16 GB)
    hard_disk: 1 TB 7200 rpm SATA
    graphic_card: NVIDIA® Quadro® P600 (4 GB GDDR5 dedicated)
    display: None
    Actual_price: 181,301
    final_price: 164,201
    Discount: 17,100
    included_items: HP Original Bag  (Worth ₹1,499) (#5DD44PA),3 Years Warranty
    --------------------------------------------------
    Name: OMEN by HP - 15-dc0084tx
    rating: 4.3333
    processor: 8th Generation Intel® Core™ i7 processor
    processor_company: Intel
    processor_type: i7
    generation: 8
    os_installed: Windows 10 Home Single Language 64
    ram: 16 GB DDR4-2666 SDRAM (1 x 16 GB)
    hard_disk: 1 TB 7200 rpm SATA + 128 GB SSD
    graphic_card: NVIDIA® GeForce® GTX 1050 Ti (4 GB GDDR5 dedicated)
    display: None
    Actual_price: 134,859
    final_price: 125,437
    Discount: 9,422
    included_items: Omen Gaming Bag (Worth ₹4,950)
    --------------------------------------------------
    Name: HP Pavilion x360 - 14-cd0051tx
    rating: 4.0000
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Home Single Language 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 1 TB 5400 rpm SATA SSHD
    graphic_card: NVIDIA® GeForce® MX130 (2 GB GDDR5 dedicated)
    display: None
    Actual_price: 82,659
    final_price: 75,814
    Discount: 6,845
    included_items: HP Original Laptop Bag (Worth ₹1,123),HP Active Pen (Worth ₹3,145),Microsoft Office Home and Student
    --------------------------------------------------
    Name: HP 348 G4 Notebook PC
    rating: 0.0000
    processor: 7th Generation Intel® Core™ i7 processor
    processor_company: Intel
    processor_type: i7
    generation: 7
    os_installed: Windows 10 Pro 64 – HP recommends Windows 10 Pro.
    ram: 8 GB DDR4-2133 SDRAM (1 x 8 GB)
    hard_disk: 1 TB 7200 rpm SATA
    graphic_card: Intel® HD Graphics 620
    display: None
    Actual_price: 74,867
    final_price: 71,153
    Discount: 3,714
    included_items: HP Original Bag  (Worth ₹1,499) (#5DD44PA)
    --------------------------------------------------
    Name: HP 348 G4 Notebook PC
    rating: 5.0000
    processor: 7th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 7
    os_installed: FreeDOS
    ram: 8 GB DDR4-2133 SDRAM (1 x 8 GB)
    hard_disk: 1 TB 7200 rpm SATA
    graphic_card: Intel® HD Graphics 620
    display: 14" diagonal HD SVA eDP anti-glare LED-backlit (1366 x 768)
    Actual_price: 55,539
    final_price: 52,783
    Discount: 2,756
    included_items: HP Original Bag  (Worth ₹1,499) (#5DD44PA)
    --------------------------------------------------
    Name: HP 348 G4 Notebook PC
    rating: 4.0000
    processor: 7th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 7
    os_installed: Windows 10 Pro 64 – HP recommends Windows 10 Pro.
    ram: 8 GB DDR4-2133 SDRAM (1 x 8 GB)
    hard_disk: 1 TB 7200 rpm SATA
    graphic_card: Intel® HD Graphics 620
    display: 14" diagonal HD SVA eDP anti-glare LED-backlit (1366 x 768)
    Actual_price: 65,907
    final_price: 62,638
    Discount: 3,269
    included_items: HP Original Bag  (Worth ₹1,499) (#5DD44PA)
    --------------------------------------------------
    Name: HP EliteBook 830 G5 Notebook PC
    rating: 5.0000
    processor: 8th Generation Intel® Core™ i7 processor
    processor_company: Intel
    processor_type: i7
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 512 GB PCIe® NVMe™ M.2 SSD
    graphic_card: Intel® UHD Graphics 620
    display: HP Sure View Integrated Privacy Screen 33.78 cm(13.3) diagonal FHD IPS anti-glare LED-backlit, 300 cd/m², 72% sRGB (1920 x 1080)
    Actual_price: 138,867
    final_price: 137,654
    Discount: 1,213
    included_items: HP Business Backpack (Worth ₹4,200)
    --------------------------------------------------
    Name: HP EliteBook x360 1030 G2
    rating: 0.0000
    processor: 7th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 7
    os_installed: Windows 10 Pro 64
    ram: 8 GB LPDDR3-1866 SDRAM (onboard)
    hard_disk: 256 GB PCIe® NVMe™ SSD
    graphic_card: Intel® HD Graphics 620
    display: None
    Actual_price: 138,867
    final_price: 137,654
    Discount: 1,213
    included_items: HP Business Backpack (Worth ₹4,200)
    --------------------------------------------------
    Name: HP EliteBook 1040 G4 Notebook PC
    rating: 0.0000
    processor: 7th Generation Intel® Core™ i7 processor
    processor_company: Intel
    processor_type: i7
    generation: 7
    os_installed: Windows 10 Pro 64
    ram: 16 GB DDR4-2133 SDRAM (onboard)
    hard_disk: 1 TB PCIe® NVMe™ SSD
    graphic_card: Intel® HD Graphics 630
    display: 35.56 cm(14) diagonal FHD IPS eDP anti-glare LED-backlit, 300 cd/m², 72% sRGB with privacy screen (1920 x 1080)
    Actual_price: 236,787
    final_price: 234,084
    Discount: 2,703
    included_items: HP Business Backpack (Worth ₹4,200)
    --------------------------------------------------
    Name: HP EliteBook x360 1030 G2 (ENERGY STAR)
    rating: 5.0000
    processor: 7th Generation Intel® Core™ i7 processor
    processor_company: Intel
    processor_type: i7
    generation: 7
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2133 SDRAM (onboard)
    hard_disk: 256 GB M.2 SATA TLC SSD
    graphic_card: Intel® HD Graphics 620
    display: None
    Actual_price: 161,267
    final_price: 159,858
    Discount: 1,409
    included_items: HP Business Backpack (Worth ₹4,200)
    --------------------------------------------------
    Name: HP ProBook 450 G6 Notebook PC
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i5 processor (i5-8265U)
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: NVIDIA® GeForce® MX130 (2 GB DDR5 dedicated)
    display: None
    Actual_price: 80,627
    final_price: 78,240
    Discount: 2,387
    included_items: HP Overnighter Backpack (Worth ₹2,499)
    --------------------------------------------------
    Name: HP ProBook 440 G6 Notebook PC
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i3 processor
    processor_company: Intel
    processor_type: i3
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 4 GB DDR4-2400 SDRAM (1 x 4 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 65,267
    final_price: 63,335
    Discount: 1,932
    included_items: HP Overnighter Backpack (Worth ₹2,499)
    --------------------------------------------------
    Name: HP ProBook 440 G6 Notebook PC
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i5 processor (i5-8265U)
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: FreeDOS 3.0
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 67,827
    final_price: 65,819
    Discount: 2,008
    included_items: HP Overnighter Backpack (Worth ₹2,499)
    --------------------------------------------------
    Name: HP ProBook 440 G6 Notebook PC
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i7 processor
    processor_company: Intel
    processor_type: i7
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 92,147
    final_price: 89,419
    Discount: 2,728
    included_items: HP Overnighter Backpack (Worth ₹2,499)
    --------------------------------------------------
    Name: HP ProBook 430 G6 Notebook PC
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i5 processor (i5-8265U)
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 79,987
    final_price: 77,619
    Discount: 2,368
    included_items: HP Overnighter Backpack (Worth ₹2,499)
    --------------------------------------------------
    Name: HP ProBook 430 G6 Notebook PC
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i7 processor
    processor_company: Intel
    processor_type: i7
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 89,587
    final_price: 86,935
    Discount: 2,652
    included_items: HP Overnighter Backpack (Worth ₹2,499)
    --------------------------------------------------
    **************************************************
    Name: HP ProBook 430 G6 Notebook PC
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i7 processor (i7-8565U)
    processor_company: Intel
    processor_type: i7
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 16 GB DDR4-2400 SDRAM (1 x 16 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 106,227
    final_price: 103,082
    Discount: 3,145
    included_items: HP Overnighter Backpack (Worth ₹2,499)
    --------------------------------------------------
    Name: HP Pavilion x360 14-dh0101tu
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i3 processor
    processor_company: Intel
    processor_type: i3
    generation: 8
    os_installed: Windows 10 Home Single Language 64
    ram: 4 GB DDR4-2400 SDRAM (1 x 4 GB)
    hard_disk: 256 GB PCIe® NVMe™ M.2 SSD
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 51,826
    final_price: 49,990
    Discount: 1,836
    included_items: HP Active Pen (Worth ₹3,145)
    --------------------------------------------------
    Name: HP Pavilion x360 14-dh0047tu
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i3 processor
    processor_company: Intel
    processor_type: i3
    generation: 8
    os_installed: Windows 10 Home Single Language 64
    ram: 4 GB DDR4-2400 SDRAM (1 x 4 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 57,179
    final_price: 54,990
    Discount: 2,189
    included_items: HP Active Pen (Worth ₹3,145)
    --------------------------------------------------
    Name: HP ZBook 15u G5 Mobile Workstation
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 512 GB PCIe® NVMe™ SSD
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 127,636
    final_price: 127,636
    Discount: 0
    included_items: 
    --------------------------------------------------
    Name: HP Pavilion x360 14-dh0045tx
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i7 processor
    processor_company: Intel
    processor_type: i7
    generation: 8
    os_installed: Windows 10 Home Single Language 64
    ram: 16 GB DDR4-2400 SDRAM (2 x 8 GB)
    hard_disk: 512 GB PCIe® NVMe™ M.2 SSD
    graphic_card: NVIDIA® GeForce® MX250 (2 GB GDDR5 dedicated)
    display: None
    Actual_price: 101,073
    final_price: 95,990
    Discount: 5,083
    included_items: HP Active Pen (Worth ₹3,145)
    --------------------------------------------------
    Name: HP Pavilion x360 14-dh0043tx
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Home Single Language 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: NVIDIA® GeForce® MX130 (2 GB GDDR5 dedicated)
    display: None
    Actual_price: 80,732
    final_price: 79,990
    Discount: 742
    included_items: HP Active Pen (Worth ₹3,145)
    --------------------------------------------------
    Name: HP Pavilion x360 14-dh0042tu
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Home Single Language 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 77,520
    final_price: 73,990
    Discount: 3,530
    included_items: HP Active Pen (Worth ₹3,145)
    --------------------------------------------------
    Name: HP Pavilion x360 14-dh0043tu
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Home Single Language 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 256 GB PCIe® NVMe™ M.2 SSD
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 71,097
    final_price: 67,990
    Discount: 3,107
    included_items: HP Active Pen (Worth ₹3,145)
    --------------------------------------------------
    Name: HP Pavilion x360 14-dh0044tx
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i3 processor
    processor_company: Intel
    processor_type: i3
    generation: 8
    os_installed: Windows 10 Home Single Language 64
    ram: 4 GB DDR4-2400 SDRAM (1 x 4 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: NVIDIA® GeForce® MX130 (2 GB GDDR5 dedicated)
    display: 14" diagonal FHD IPS anti-glare micro-edge WLED-backlit touch screen (1920 x 1080)
    Actual_price: 62,532
    final_price: 59,990
    Discount: 2,542
    included_items: HP Active Pen (Worth ₹3,145)
    --------------------------------------------------
    Name: HP Notebook - 15-da0073tx
    rating: 0.0000
    processor: 7th Generation Intel® Core™ i3 processor
    processor_company: Intel
    processor_type: i3
    generation: 7
    os_installed: FreeDOS 2.0
    ram: 4 GB DDR4-2133 SDRAM (1 x 4 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: NVIDIA® GeForce® MX110 (2 GB DDR3 dedicated)
    display: 39.62 cm(15.6) diagonal FHD SVA anti-glare micro-edge WLED-backlit (1920 x 1080)
    Actual_price: 40,458
    final_price: 35,507
    Discount: 4,951
    included_items: HP Original Laptop Bag (Worth ₹1,123),1 Year On-site Warranty
    --------------------------------------------------
    Name: HP ZBook 14u G5 Mobile Workstation
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Pro
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 512 GB HP Z Turbo Drive PCIe® SSD
    graphic_card: AMD Radeon™ Pro WX 3100 Graphics (2 GB GDDR5 dedicated)
    display: None
    Actual_price: 109,386
    final_price: 109,386
    Discount: 0
    included_items: 
    --------------------------------------------------
    Name: HP Notebook - 15-da0447tx
    rating: 4.0000
    processor: 7th Generation Intel® Core™ i3 processor
    processor_company: Intel
    processor_type: i3
    generation: 7
    os_installed: Windows 10 Home Single Language 64
    ram: 4 GB DDR4-2133 SDRAM (1 x 4 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: NVIDIA® GeForce® MX110 (2 GB GDDR5 dedicated)
    display: 15.6" diagonal FHD SVA anti-glare WLED-backlit (1920 x 1080)
    Actual_price: 42,369
    final_price: 41,111
    Discount: 1,258
    included_items: HP Original Laptop Bag (Worth ₹1,123)
    --------------------------------------------------
    Name: HP 240 G6 Notebook PC
    rating: 2.0000
    processor: 7th Generation Intel® Core™ i3 processor
    processor_company: Intel
    processor_type: i3
    generation: 7
    os_installed: Windows 10 Home Single Language 64 – HP recommends Windows 10 Pro.
    ram: 4 GB DDR4-2133 SDRAM (1 x 4 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 35,630
    final_price: 34,050
    Discount: 1,580
    included_items: HP Original Bag  (Worth ₹1,499) (#5DD44PA)
    --------------------------------------------------
    Name: HP 250 G6 Notebook PC
    rating: 0.0000
    processor: 7th Generation Intel® Core™ i3 processor
    processor_company: Intel
    processor_type: i3
    generation: 7
    os_installed: Windows 10 Home Single Language 64 – HP recommends Windows 10 Pro.
    ram: 4 GB DDR4-2133 SDRAM (1 x 4 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: Intel® HD Graphics 620
    display: None
    Actual_price: 36,686
    final_price: 35,033
    Discount: 1,653
    included_items: HP Original Bag  (Worth ₹1,499) (#5DD44PA)
    --------------------------------------------------
    Name: HP Spectre x360 - 13-ap0122tu
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i7 processor
    processor_company: Intel
    processor_type: i7
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 16 GB DDR4-2400 SDRAM (onboard)
    hard_disk: 512 GB PCIe® NVMe™ M.2 SSD
    graphic_card: Intel® UHD Graphics 620
    display: HP Sure View Integrated Privacy Screen 13.3" diagonal FHD IPS anti-glare micro-edge WLED-backlit touch screen (1920 x 1080)
    Actual_price: 176,639
    final_price: 170,875
    Discount: 5,764
    included_items: 
    --------------------------------------------------
    Name: HP Spectre x360 - 13-ap0121tu
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2400 SDRAM (onboard)
    hard_disk: 256 GB PCIe® NVMe™ M.2 SSD
    graphic_card: Intel® UHD Graphics 620
    display: HP Sure View Integrated Privacy Screen 13.3" diagonal FHD IPS anti-glare micro-edge WLED-backlit touch screen (1920 x 1080)
    Actual_price: 144,521
    final_price: 140,875
    Discount: 3,646
    included_items: 
    --------------------------------------------------
    Name: HP Spectre x360 - 13-ap0102tu
    rating: 5.0000
    processor: 8th Generation Intel® Core™ i7 processor
    processor_company: Intel
    processor_type: i7
    generation: 8
    os_installed: Windows 10 Home Single Language 64
    ram: 16 GB DDR4-2400 SDRAM (onboard)
    hard_disk: 1 TB PCIe® NVMe™ M.2 SSD
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 203,404
    final_price: 190,874
    Discount: 12,530
    included_items: Microsoft Office Home and Student
    --------------------------------------------------
    Name: HP Spectre x360 - 13-ap0101tu
    rating: 2.0000
    processor: 8th Generation Intel® Core™ i7 processor
    processor_company: Intel
    processor_type: i7
    generation: 8
    os_installed: Windows 10 Home Single Language 64
    ram: 16 GB DDR4-2400 SDRAM (onboard)
    hard_disk: 512 GB PCIe® NVMe™ M.2 SSD
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 171,286
    final_price: 160,875
    Discount: 10,411
    included_items: Microsoft Office Home and Student
    --------------------------------------------------
    Name: HP Spectre x360 - 13-ap0100tu
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Home Single Language 64
    ram: 8 GB DDR4-2400 SDRAM (onboard)
    hard_disk: 256 GB PCIe® NVMe™ M.2 SSD
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 139,168
    final_price: 130,874
    Discount: 8,294
    included_items: Microsoft Office Home and Student
    --------------------------------------------------
    Name: HP ZBook 14u G5 Mobile Workstation
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i7 processor (i7-8650U, i7-8550U)
    processor_company: Intel
    processor_type: i7
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 512 GB HP Z Turbo Drive PCIe® SSD
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 160,151
    final_price: 127,767
    Discount: 32,384
    included_items: 
    --------------------------------------------------
    Name: HP 240 G7 Notebook PC
    rating: 0.0000
    processor: 7th Generation Intel® Core™ i3 processor
    processor_company: Intel
    processor_type: i3
    generation: 7
    os_installed: FreeDOS 2.0
    ram: 4 GB DDR4-2133 SDRAM (1 x 4 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: Intel® HD Graphics 620
    display: None
    Actual_price: 37,107
    final_price: 35,267
    Discount: 1,840
    included_items: HP Original Bag  (Worth ₹1,499) (#5DD44PA)
    --------------------------------------------------
    Name: HP 240 G7 Notebook PC
    rating: 0.0000
    processor: 7th Generation Intel® Core™ i3 processor (i3-7020U)
    processor_company: Intel
    processor_type: i3
    generation: 7
    os_installed: Windows 10 Pro 64
    ram: 4 GB DDR4-2133 SDRAM (1 x 4 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: Intel® HD Graphics 620
    display: None
    Actual_price: 50,291
    final_price: 47,796
    Discount: 2,495
    included_items: HP Original Bag  (Worth ₹1,499) (#5DD44PA)
    --------------------------------------------------
    Name: HP 240 G7 Notebook PC
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i5 processor (i5-8250U)
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: FreeDOS 2.0
    ram: 4 GB DDR4-2400 SDRAM (1 x 4 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 49,907
    final_price: 47,431
    Discount: 2,476
    included_items: HP Original Bag  (Worth ₹1,499) (#5DD44PA)
    --------------------------------------------------
    Name: HP 240 G7 Notebook PC
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i5 processor (i5-8250U)
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 63,987
    final_price: 60,812
    Discount: 3,175
    included_items: HP Original Bag  (Worth ₹1,499) (#5DD44PA)
    --------------------------------------------------
    Name: HP 348 G4 Notebook PC (ENERGY STAR)
    rating: 0.0000
    processor: 7th Generation Intel® Core™ i3 processor (i3-7020U)
    processor_company: Intel
    processor_type: i3
    generation: 7
    os_installed: Windows 10 Pro 64
    ram: 4 GB DDR4-2133 SDRAM (1 x 4 GB)
    hard_disk: 1 TB 7200 rpm SATA
    graphic_card: Intel® HD Graphics 620
    display: None
    Actual_price: 51,827
    final_price: 49,256
    Discount: 2,571
    included_items: HP Original Bag  (Worth ₹1,499) (#5DD44PA)
    --------------------------------------------------
    Name: HP ZBook 17 G5 Mobile Workstation
    rating: 0.0000
    processor: Intel® Xeon® processor
    processor_company: Intel
    processor_type: Xeon®
    generation: 7
    os_installed: Windows 10 Pro 64
    ram: 32 GB DDR4-2666 ECC SDRAM (2 X 16 GB)
    hard_disk: 512 GB PCIe® NVMe™ SSD
    graphic_card: NVIDIA® Quadro® P4200 (8 GB GDDR5 dedicated)
    display: None
    Actual_price: 667,152
    final_price: 442,671
    Discount: 224,481
    included_items: 
    --------------------------------------------------
    Name: HP ZBook 15v G5 Mobile Workstation
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i7 processor
    processor_company: Intel
    processor_type: i7
    generation: 8
    os_installed: FreeDOS 2.0
    ram: 8 GB DDR4-2666 SDRAM (1 x 8 GB)
    hard_disk: None
    graphic_card: NVIDIA® Quadro® P600 (4 GB GDDR5 dedicated)
    display: None
    Actual_price: 183,207
    final_price: 117,184
    Discount: 66,023
    included_items: 
    --------------------------------------------------
    Name: HP ZBook 15u G5 Mobile Workstation
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i7 processor
    processor_company: Intel
    processor_type: i7
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 512 GB PCIe® NVMe™ SSD
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 162,354
    final_price: 125,210
    Discount: 37,144
    included_items: 
    --------------------------------------------------
    Name: HP ZBook 15u G5 Mobile Workstation
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i7 processor
    processor_company: Intel
    processor_type: i7
    generation: 8
    os_installed: FreeDOS 2.0
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 512 GB PCIe® NVMe™ SSD
    graphic_card: Intel® HD Graphics 630
    display: None
    Actual_price: 169,300
    final_price: 112,694
    Discount: 56,606
    included_items: 
    --------------------------------------------------
    Name: HP ZBook 15u G5 Mobile Workstation
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: FreeDOS 2.0
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 512 GB PCIe® NVMe™ SSD
    graphic_card: AMD Radeon™ Pro WX 3100 Graphics (2 GB GDDR5 dedicated)
    display: None
    Actual_price: 166,690
    final_price: 109,386
    Discount: 57,304
    included_items: 
    --------------------------------------------------
    **************************************************
    Name: HP EliteBook x360 1030 G3 Notebook PC
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2133 SDRAM (onboard)
    hard_disk: 512 GB SSD
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 159,347
    final_price: 157,955
    Discount: 1,392
    included_items: HP Active Pen (Worth ₹3,145),HP Original Bag  (Worth ₹1,499) (#5DD44PA)
    --------------------------------------------------
    Name: HP EliteBook x360 1030 G3 Notebook PC
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i7 processor
    processor_company: Intel
    processor_type: i7
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 16 GB DDR4-2133 SDRAM (onboard)
    hard_disk: 1 TB SSD
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 204,403
    final_price: 202,618
    Discount: 1,785
    included_items: HP Active Pen (Worth ₹3,145),HP Original Bag  (Worth ₹1,499) (#5DD44PA)
    --------------------------------------------------
    Name: HP ZBook 14u G5 Mobile Workstation
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i7 processor
    processor_company: Intel
    processor_type: i7
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 16 GB DDR4-2400 SDRAM (1 x 16 GB)
    hard_disk: 512 GB SATA SSD
    graphic_card: AMD Radeon™ Pro WX 3100 Graphics (2 GB GDDR5 dedicated)
    display: None
    Actual_price: 275,830
    final_price: 161,517
    Discount: 114,313
    included_items: 
    --------------------------------------------------
    Name: HP ZBook 15u G5 Mobile Workstation
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 512 GB SSD
    graphic_card: AMD Radeon™ Pro WX 3100 Graphics (2 GB GDDR5 dedicated)
    display: None
    Actual_price: 206,659
    final_price: 126,274
    Discount: 80,385
    included_items: 
    --------------------------------------------------
    Name: HP ZBook 14u G5 Mobile Workstation
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i7 processor
    processor_company: Intel
    processor_type: i7
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 16 GB DDR4-2400 SDRAM (1 x 16 GB)
    hard_disk: 512 GB SATA SSD
    graphic_card: AMD Radeon™ Pro WX 3100 Graphics (2 GB GDDR5 dedicated)
    display: None
    Actual_price: 252,946
    final_price: 150,769
    Discount: 102,177
    included_items: 
    --------------------------------------------------
    Name: HP ZBook 14u G5 Mobile Workstation
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i7 processor
    processor_company: Intel
    processor_type: i7
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 512 GB SATA SSD
    graphic_card: AMD Radeon™ Pro WX 3100 Graphics (2 GB GDDR5 dedicated)
    display: None
    Actual_price: 237,270
    final_price: 142,443
    Discount: 94,827
    included_items: 
    --------------------------------------------------
    Name: HP ZBook 14u G5 Mobile Workstation
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 512 GB SATA SSD
    graphic_card: AMD Radeon™ Pro WX 3100 Graphics (2 GB GDDR5 dedicated)
    display: None
    Actual_price: 217,955
    final_price: 126,274
    Discount: 91,681
    included_items: 
    --------------------------------------------------
    Name: HP ZBook 15u G5 Mobile Workstation
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i7 processor
    processor_company: Intel
    processor_type: i7
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 512 GB PCIe® NVMe™ SSD
    graphic_card: AMD Radeon™ Pro WX 3100 Graphics (2 GB GDDR5 dedicated)
    display: None
    Actual_price: 249,071
    final_price: 153,040
    Discount: 96,031
    included_items: 
    --------------------------------------------------
    Name: HP ZBook 14u G5 Mobile Workstation
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i7 processor
    processor_company: Intel
    processor_type: i7
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 512 GB SATA SSD
    graphic_card: AMD Radeon™ Pro WX 3100 Graphics (2 GB GDDR5 dedicated)
    display: None
    Actual_price: 260,367
    final_price: 153,040
    Discount: 107,327
    included_items: 
    --------------------------------------------------
    Name: HP ProBook 440 G5 Notebook PC
    rating: 0.0000
    processor: 7th Generation Intel® Core™ i3 processor
    processor_company: Intel
    processor_type: i3
    generation: 7
    os_installed: Windows 10 Pro 64
    ram: 4 GB DDR4-2400 SDRAM (1 x 4 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: Intel® HD Graphics 620
    display: None
    Actual_price: 64,627
    final_price: 60,479
    Discount: 4,148
    included_items: HP Business Backpack (Worth ₹4,200)
    --------------------------------------------------
    Name: HP ProBook 430 G5 Notebook PC
    rating: 0.0000
    processor: 7th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 7
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 512 GB PCIe® NVMe™ M.2 SSD
    graphic_card: Intel® HD Graphics 620
    display: None
    Actual_price: 97,779
    final_price: 94,885
    Discount: 2,894
    included_items: HP Business Backpack (Worth ₹4,200)
    --------------------------------------------------
    Name: HP EliteBook 735 G5 Notebook PC
    rating: 0.0000
    processor: AMD® Ryzen™ APU processor
    processor_company: AMD
    processor_type: Ryzen™ APU
    generation: None
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 512 GB SSD
    graphic_card: AMD Radeon™ Vega Graphics
    display: None
    Actual_price: 106,227
    final_price: 105,299
    Discount: 928
    included_items: HP Business Backpack (Worth ₹4,200)
    --------------------------------------------------
    Name: HP EliteBook 1050 G1 Notebook PC
    rating: 5.0000
    processor: 8th Generation Intel® Core™ i7 processor (i7-8750H)
    processor_company: Intel
    processor_type: i7
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 16 GB DDR4-2666 SDRAM (1 x 16 GB)
    hard_disk: 1 TB SSD
    graphic_card: NVIDIA® GeForce® GTX 1050 (4 GB DDR5 dedicated)
    display: None
    Actual_price: 245,747
    final_price: 243,600
    Discount: 2,147
    included_items: HP Business Backpack (Worth ₹4,200)
    --------------------------------------------------
    Name: HP ZBook 15u G5 Mobile Workstation
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i7 processor
    processor_company: Intel
    processor_type: i7
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 16 GB DDR4-2400 SDRAM (1 x 16 GB)
    hard_disk: 512 GB PCIe® NVMe™ SSD
    graphic_card: AMD Radeon™ Pro WX 3100 Graphics (2 GB GDDR5 dedicated)
    display: None
    Actual_price: 241,510
    final_price: 150,769
    Discount: 90,741
    included_items: HP Original Bag  (Worth ₹1,499) (#5DD44PA),3 Years Warranty
    --------------------------------------------------
    Name: HP ZBook 15u G5 Mobile Workstation
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i7 processor
    processor_company: Intel
    processor_type: i7
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 512 GB PCIe® NVMe™ SSD
    graphic_card: AMD Radeon™ Pro WX 3100 Graphics (2 GB GDDR5 dedicated)
    display: None
    Actual_price: 225,835
    final_price: 142,443
    Discount: 83,392
    included_items: HP Business Backpack (Worth ₹4,200)
    --------------------------------------------------
    Name: HP ZBook Studio G5 Mobile Workstation
    rating: 3.0000
    processor: 8th Generation Intel® Core™ i7 processor
    processor_company: Intel
    processor_type: i7
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 16 GB DDR4-2666 SDRAM (1 x 16 GB)
    hard_disk: 512 GB PCIe® NVMe™ SSD
    graphic_card: NVIDIA® Quadro® P1000 (4 GB GDDR5 dedicated)
    display: None
    Actual_price: 314,632
    final_price: 221,580
    Discount: 93,052
    included_items: HP Business Backpack (Worth ₹4,200)
    --------------------------------------------------
    Name: HP ZBook 15 G5 Mobile Workstation
    rating: 0.0000
    processor: None
    processor_company: None
    processor_type: None
    generation: None
    os_installed: Windows 10 Pro 64
    ram: 16 GB (1x16 GB) DDR4 2666
    hard_disk: 1 TB 7200 rpm SATA + 512 GB SSD
    graphic_card: NVIDIA® Quadro® P2000 (4 GB GDDR5 dedicated)
    display: None
    Actual_price: 350,406
    final_price: 225,553
    Discount: 124,853
    included_items: HP Business Backpack (Worth ₹4,200)
    --------------------------------------------------
    Name: HP ZBook 15 G5 Mobile Workstation
    rating: 0.0000
    processor: None
    processor_company: None
    processor_type: None
    generation: None
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2666 SDRAM (1 x 8 GB)
    hard_disk: 1 TB 7200 rpm SATA + 128 GB SSD
    graphic_card: NVIDIA® Quadro® P1000 (4 GB GDDR5 dedicated)
    display: None
    Actual_price: 316,190
    final_price: 205,419
    Discount: 110,771
    included_items: HP Business Backpack (Worth ₹4,200)
    --------------------------------------------------
    Name: HP ZBook 15 G5 Mobile Workstation
    rating: 0.0000
    processor: None
    processor_company: None
    processor_type: None
    generation: None
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2666 SDRAM (1 x 8 GB)
    hard_disk: 1 TB 7200 rpm SATA
    graphic_card: NVIDIA® Quadro® P1000 (4 GB GDDR5 dedicated)
    display: None
    Actual_price: 240,765
    final_price: 156,824
    Discount: 83,941
    included_items: HP Business Backpack (Worth ₹4,200)
    --------------------------------------------------
    Name: HP ZBook x2 G4 Detachable Workstation
    rating: 0.0000
    processor: None
    processor_company: None
    processor_type: None
    generation: None
    os_installed: Windows 10 Pro 64
    ram: 16 GB DDR4-2133 SDRAM (2 x 8 GB)
    hard_disk: 512 GB SSD
    graphic_card: NVIDIA® Quadro® M620 (2 GB GDDR5 dedicated)
    display: None
    Actual_price: 408,283
    final_price: 262,386
    Discount: 145,897
    included_items: HP Business Backpack (Worth ₹4,200)
    --------------------------------------------------
    Name: HP ZBook x2 G4 Detachable Workstation
    rating: 0.0000
    processor: None
    processor_company: None
    processor_type: None
    generation: None
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2133 SDRAM (2 x 4 GB)
    hard_disk: 512 GB SSD
    graphic_card: NVIDIA® Quadro® M620 (2 GB GDDR5 dedicated)
    display: None
    Actual_price: 369,052
    final_price: 240,692
    Discount: 128,360
    included_items: HP Business Backpack (Worth ₹4,200)
    --------------------------------------------------
    Name: HP ZBook Studio x360 G5 Convertible Workstation
    rating: 0.0000
    processor: None
    processor_company: None
    processor_type: None
    generation: None
    os_installed: Windows 10 Pro 64
    ram: 16 GB DDR4-2666 SDRAM (1 x 16 GB)
    hard_disk: 1 TB SSD
    graphic_card: NVIDIA® Quadro® P1000 (4 GB GDDR5 dedicated)
    display: None
    Actual_price: 400,665
    final_price: 256,285
    Discount: 144,380
    included_items: HP Business Backpack (Worth ₹4,200)
    --------------------------------------------------
    Name: HP ZBook Studio x360 G5 Convertible Workstation
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2666 SDRAM (1 x 8 GB)
    hard_disk: 512 GB PCIe® NVMe™ SSD
    graphic_card: NVIDIA® Quadro® P1000 (4 GB GDDR5 dedicated)
    display: None
    Actual_price: 315,831
    final_price: 215,456
    Discount: 100,375
    included_items: HP Business Backpack (Worth ₹4,200)
    --------------------------------------------------
    Name: HP ZBook Studio G5 Mobile Workstation
    rating: 0.0000
    processor: None
    processor_company: None
    processor_type: None
    generation: None
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2666 SDRAM (1 x 8 GB)
    hard_disk: 512 GB SSD
    graphic_card: NVIDIA® Quadro® P1000 (4 GB GDDR5 dedicated)
    display: None
    Actual_price: 254,780
    final_price: 164,999
    Discount: 89,781
    included_items: HP Business Backpack (Worth ₹4,200)
    --------------------------------------------------
    Name: HP Notebook - 15-da0296tu
    rating: 4.0000
    processor: 7th Generation Intel® Core™ i3 processor
    processor_company: Intel
    processor_type: i3
    generation: 7
    os_installed: FreeDOS 2.0
    ram: 4 GB DDR4-2133 SDRAM (1 x 4 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: Intel® HD Graphics 620
    display: None
    Actual_price: 35,197
    final_price: 31,911
    Discount: 3,286
    included_items: HP Original Laptop Bag (Worth ₹1,123)
    --------------------------------------------------
    Name: HP Notebook - 15-da0074tx
    rating: 5.0000
    processor: 7th Generation Intel® Core™ i3 processor
    processor_company: Intel
    processor_type: i3
    generation: 7
    os_installed: FreeDOS 2.0
    ram: 8 GB DDR4-2133 SDRAM (1 x 8 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: NVIDIA® GeForce® MX110 (2 GB DDR3 dedicated)
    display: None
    Actual_price: 46,010
    final_price: 39,494
    Discount: 6,516
    included_items: HP Original Laptop Bag (Worth ₹1,123)
    --------------------------------------------------
    Name: HP Notebook - 15-da0300tu
    rating: 5.0000
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: FreeDOS 2.0
    ram: 4 GB DDR4-2400 SDRAM (1 x 4 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 47,294
    final_price: 40,551
    Discount: 6,743
    included_items: HP Original Laptop Bag (Worth ₹1,123)
    --------------------------------------------------
    Name: HP Notebook - 15-da0077tx
    rating: 5.0000
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: FreeDOS 2.0
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: NVIDIA® GeForce® MX110 (2 GB GDDR5 dedicated)
    display: None
    Actual_price: 57,786
    final_price: 50,721
    Discount: 7,065
    included_items: HP Original Laptop Bag (Worth ₹1,123),1 Year On-site Warranty
    --------------------------------------------------
    Name: HP ENVY - 13-ah0044tx
    rating: 4.0000
    processor: 8th Generation Intel® Core™ i7 processor
    processor_company: Intel
    processor_type: i7
    generation: 8
    os_installed: Windows 10 Home Single Language 64
    ram: 8 GB LPDDR3-2133 SDRAM (onboard)
    hard_disk: 256 GB SSD
    graphic_card: NVIDIA® GeForce® MX150 (2 GB GDDR5 dedicated)
    display: None
    Actual_price: 102,232
    final_price: 96,990
    Discount: 5,242
    included_items: 
    --------------------------------------------------
    Name: HP ENVY - 13-ah0043tx
    rating: 5.0000
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Home Single Language 64
    ram: 8 GB LPDDR3-2133 SDRAM (onboard)
    hard_disk: 256 GB SSD
    graphic_card: NVIDIA® GeForce® MX150 (2 GB GDDR5 dedicated)
    display: None
    Actual_price: 88,742
    final_price: 84,390
    Discount: 4,352
    included_items: 1 Year On-site Warranty
    --------------------------------------------------
    **************************************************
    Name: HP ENVY - 13-ah0042tu
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i3 processor
    processor_company: Intel
    processor_type: i3
    generation: 8
    os_installed: Windows 10 Home Single Language 64
    ram: 4 GB LPDDR3-1866 SDRAM (onboard)
    hard_disk: 128 GB SSD
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 64,011
    final_price: 60,380
    Discount: 3,631
    included_items: 1 Year On-site Warranty
    --------------------------------------------------
    Name: HP EliteBook 830 G5 Notebook PC
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 256 GB SSD
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 107,507
    final_price: 106,568
    Discount: 939
    included_items: 
    --------------------------------------------------
    Name: HP ProBook 640 G4 Notebook PC
    rating: 0.0000
    processor: None
    processor_company: None
    processor_type: None
    generation: None
    os_installed: 8th Generation Intel® Core™ i5 processor
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 1 TB 7200 rpm SATA
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 95,603
    final_price: 92,773
    Discount: 2,830
    included_items: HP Overnighter Backpack (Worth ₹2,499)
    --------------------------------------------------
    Name: HP ProBook 640 G4 Notebook PC
    rating: 0.0000
    processor: None
    processor_company: None
    processor_type: None
    generation: None
    os_installed: Intel Core i7-8550U Processor
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 1 TB 7200 rpm SATA
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 106,867
    final_price: 103,703
    Discount: 3,164
    included_items: HP Overnighter Backpack (Worth ₹2,499)
    --------------------------------------------------
    Name: HP 348 G3 Notebook PC
    rating: 0.0000
    processor: 6th Generation Intel® Core™ i3 processor
    processor_company: Intel
    processor_type: i3
    generation: 6
    os_installed: Windows 10 Pro 64
    ram: 4 GB DDR4-2133 SDRAM (1 x 4 GB)
    hard_disk: 1 TB 7200 rpm SATA
    graphic_card: Intel® HD Graphics 520
    display: None
    Actual_price: 51,827
    final_price: 49,256
    Discount: 2,571
    included_items: HP Original Bag  (Worth ₹1,499) (#5DD44PA)
    --------------------------------------------------
    Name: HP ProBook x360 440 G1 Notebook PC
    rating: 1.0000
    processor: 8th Generation Intel® Core™ i7 processor
    processor_company: Intel
    processor_type: i7
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 512 GB SSD
    graphic_card: Intel® UHD Graphics 620
    display: 14" FHD Touchscreen
    Actual_price: 116,211
    final_price: 112,771
    Discount: 3,440
    included_items: HP Overnighter Backpack (Worth ₹2,499)
    --------------------------------------------------
    Name: HP ZBook 15u G4 Mobile Workstation
    rating: 0.0000
    processor: 16 GB DDR4-2133 SDRAM (1 x 16 GB)
    processor_company: None
    processor_type: None
    generation: None
    os_installed: Windows 10 Pro 64
    ram: None
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: AMD FirePro™ W4190M Graphics (2 GB GDDR5 dedicated)
    display: None
    Actual_price: 176,453
    final_price: 171,000
    Discount: 5,453
    included_items: 
    --------------------------------------------------
    Name: HP 245 G5 Notebook PC
    rating: 3.5000
    processor: AMD A6 APU
    processor_company: AMD
    processor_type: A6 APU
    generation: None
    os_installed: Free DOS
    ram: 4 GB DDR3L-1600 SDRAM (1 x 4 GB)
    hard_disk: 500 GB 5400 rpm SATA
    graphic_card: AMD Radeon™ R4
    display: None
    Actual_price: 20,999
    final_price: 20,999
    Discount: 0
    included_items: 
    --------------------------------------------------
    Name: HP ProBook 440 G5 Notebook PC
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 4 GB DDR4-2400 SDRAM (1 x 4 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: Intel® UHD Graphics 620
    display: 35.56 cm(14) diagonal HD LED-backlit touch screen (1366 x 768)
    Actual_price: 83,699
    final_price: 81,222
    Discount: 2,477
    included_items: HP Overnighter Backpack (Worth ₹2,499)
    --------------------------------------------------
    Name: HP ProBook 430 G5 Notebook PC
    rating: 1.0000
    processor: 8th Generation Intel® Core™ i7 processor
    processor_company: Intel
    processor_type: i7
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: Intel® UHD graphics 620
    display: None
    Actual_price: 87,923
    final_price: 85,320
    Discount: 2,603
    included_items: HP Overnighter Backpack (Worth ₹2,499)
    --------------------------------------------------
    Name: HP 348 G4 Notebook PC
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 1 TB 7200 rpm SATA
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 65,907
    final_price: 62,637
    Discount: 3,270
    included_items: HP Original Bag  (Worth ₹1,499) (#5DD44PA)
    --------------------------------------------------
    Name: HP ProBook 450 G6 Notebook PC
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i7 processor
    processor_company: Intel
    processor_type: i7
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: NVIDIA® GeForce® MX130 (2 GB DDR5 dedicated)
    display: None
    Actual_price: 92,147
    final_price: 89,419
    Discount: 2,728
    included_items: HP Overnighter Backpack (Worth ₹2,499)
    --------------------------------------------------
    Name: HP ProBook 440 G6 Notebook PC
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i5 processor (i5-8265U)
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 83,187
    final_price: 80,725
    Discount: 2,462
    included_items: HP Overnighter Backpack (Worth ₹2,499)
    --------------------------------------------------
    Name: HP ProBook 430 G6 Notebook PC
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i5 processor (i5-8265U)
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 86,387
    final_price: 83,830
    Discount: 2,557
    included_items: HP Overnighter Backpack (Worth ₹2,499)
    --------------------------------------------------
    something went wrong
    Name: HP Elite x2 1012 G1 Tablet (ENERGY STAR)
    rating: 0.0000
    processor: 6th Generation Intel® Core™ m5 processor
    processor_company: Intel
    processor_type: i5
    generation: 6
    os_installed: Windows 10 Pro 64
    ram: 4 GB LPDDR3-1866 SDRAM
    hard_disk: 128 GB M.2 SATA TLC SSD
    graphic_card: Intel® HD Graphics 515
    display: 12" diagonal FHD UWVA eDP ultra-slim LED-backlit touch screen (1920 x1280), direct bonded, Corning® Gorilla® Glass 4
    Actual_price: 136,356
    final_price: 85,000
    Discount: 51,356
    included_items: Free HP Business Top Load Case (Worth ₹1,775) (#H5M92AA)
    --------------------------------------------------
    Name: HP Chromebook 11A G6 EE
    rating: 0.0000
    processor: AMD A4-Series APU processor
    processor_company: AMD
    processor_type: A4-Series APU
    generation: None
    os_installed: Chrome OS™ 64
    ram: 4 GB DDR4-2666 SDRAM (onboard)
    hard_disk: 16 GB eMMC
    graphic_card: AMD Radeon™ R4 Graphics
    display: None
    Actual_price: 28,320
    final_price: 28,320
    Discount: 0
    included_items: 
    --------------------------------------------------
    Name: OMEN by HP 15-dc1006tx
    rating: 5.0000
    processor: 8th Generation Intel® Core™ i7 processor
    processor_company: Intel
    processor_type: i7
    generation: 8
    os_installed: Windows 10 Home 64
    ram: 16 GB DDR4-2666 SDRAM (1 x 16 GB)
    hard_disk: None
    graphic_card: NVIDIA® GeForce® RTX 2060 Graphics (6 GB GDDR6 dedicated)
    display: None
    Actual_price: 158,841
    final_price: 151,875
    Discount: 6,966
    included_items: Omen Gaming Bag (Worth ₹4,950)
    --------------------------------------------------
    Name: HP 240 G6 Notebook PC
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 4 GB DDR4-2400 SDRAM (1 x 4 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 60,019
    final_price: 57,041
    Discount: 2,978
    included_items: HP Original Bag  (Worth ₹1,499) (#5DD44PA)
    --------------------------------------------------
    Name: HP 240 G6 Notebook PC
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: FreeDOS 2.0
    ram: 4 GB DDR4-2400 SDRAM (1 x 4 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 49,907
    final_price: 47,431
    Discount: 2,476
    included_items: HP Original Bag  (Worth ₹1,499) (#5DD44PA)
    --------------------------------------------------
    Name: HP Spectre Folio - 13-ak0040tu
    rating: 5.0000
    processor: 8th Generation Intel® Core™ i7 processor
    processor_company: Intel
    processor_type: i7
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 16 GB LPDDR3-1866 SDRAM (onboard)
    hard_disk: 512 GB PCIe® NVMe™ M.2 SSD
    graphic_card: Intel® UHD Graphics 615
    display: 13.3" diagonal FHD IPS BrightView WLED-backlit micro-edge multitouch-enabled edge-to-edge glass (1920 x 1080)
    Actual_price: 214,110
    final_price: 200,875
    Discount: 13,235
    included_items: 
    --------------------------------------------------
    Name: HP 250 G6 Notebook PC
    rating: 0.0000
    processor: 7th Generation Intel® Core™ i3 processor
    processor_company: Intel
    processor_type: i3
    generation: 7
    os_installed: Windows 10 Home Single Language 64 – HP recommends Windows 10 Pro.
    ram: 4 GB DDR4-2133 SDRAM (1 x 4 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: Intel® HD Graphics 620
    display: None
    Actual_price: 37,200
    final_price: 37,200
    Discount: 0
    included_items: 
    --------------------------------------------------
    Name: HP ZBook Studio x360 G5 Convertible Workstation
    rating: 0.0000
    processor: Intel® Xeon® processor
    processor_company: Intel
    processor_type: Xeon®
    generation: 7
    os_installed: Windows 10 Pro 64
    ram: 16 GB DDR4-2666 SDRAM (1 x 16 GB)
    hard_disk: 512 GB PCIe® NVMe™ SSD
    graphic_card: NVIDIA® Quadro® P1000 (4 GB GDDR5 dedicated)
    display: None
    Actual_price: 437,143
    final_price: 341,440
    Discount: 95,703
    included_items: 
    --------------------------------------------------
    Name: HP ZBook 17 G5 Mobile Workstation
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i7 processor
    processor_company: Intel
    processor_type: i7
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 16 GB DDR4-2666 SDRAM (1 x 16 GB)
    hard_disk: 512 GB PCIe® NVMe™ SSD
    graphic_card: NVIDIA® Quadro® P1000 (4 GB GDDR5 dedicated)
    display: None
    Actual_price: 353,597
    final_price: 237,521
    Discount: 116,076
    included_items: 
    --------------------------------------------------
    Name: HP ZBook 15v G5 Mobile Workstation
    rating: 0.0000
    processor: Intel® Xeon® E3 processor
    processor_company: Intel
    processor_type: Xeon® E3
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 16 GB DDR4-2666 SDRAM (1 x 16 GB)
    hard_disk: 512GB PCIe NVMe TLC SSD
    graphic_card: NVIDIA® Quadro® P600 (4 GB GDDR5 dedicated)
    display: None
    Actual_price: 337,097
    final_price: 239,551
    Discount: 97,546
    included_items: 
    --------------------------------------------------
    Name: HP ZBook 15 G5 Mobile Workstation
    rating: 0.0000
    processor: Intel® Xeon® processor
    processor_company: Intel
    processor_type: Xeon®
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 16 GB DDR4-2666 SDRAM (1 x 16 GB)
    hard_disk: 512 GB PCIe® NVMe™ SSD
    graphic_card: NVIDIA® Quadro® P2000 (4 GB GDDR5 dedicated)
    display: None
    Actual_price: 347,045
    final_price: 270,969
    Discount: 76,076
    included_items: 
    --------------------------------------------------
    Name: HP ZBook 14u G5 Mobile Workstation
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i7 processor
    processor_company: Intel
    processor_type: i7
    generation: 8
    os_installed: FreeDOS 2.0
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 512 GB HP Z Turbo Drive PCIe® SSD
    graphic_card: Intel® HD Graphics 620
    display: None
    Actual_price: 164,746
    final_price: 112,694
    Discount: 52,052
    included_items: 
    --------------------------------------------------
    Name: HP 245 G6
    rating: 0.0000
    processor: BU IDS UMA A9-9425
    processor_company: None
    processor_type: None
    generation: None
    os_installed: FreeDOS
    ram: 4GB (1x4GB) 1866 DDR4
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: None
    display: None
    Actual_price: 24,200
    final_price: 24,200
    Discount: 0
    included_items: 
    --------------------------------------------------
    Name: HP EliteBook x360 1030 G3 Notebook PC
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2133 SDRAM (onboard)
    hard_disk: 256 GB SSD
    graphic_card: Intel® UHD Graphics 620
    display: None
    Actual_price: 143,347
    final_price: 142,094
    Discount: 1,253
    included_items: HP Active Pen (Worth ₹3,145),HP Original Bag  (Worth ₹1,499) (#5DD44PA)
    --------------------------------------------------
    Name: HP 250 G6 Notebook PC
    rating: 0.0000
    processor: 7th Generation Intel® Core™ i3 processor
    processor_company: Intel
    processor_type: i3
    generation: 7
    os_installed: FreeDOS 2.0
    ram: 4 GB DDR4-2133 SDRAM (1 x 4 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: AMD Radeon™ 520 Graphics (2 GB GDDR5 dedicated)
    display: None
    Actual_price: 40,947
    final_price: 38,915
    Discount: 2,032
    included_items: HP Original Bag  (Worth ₹1,499) (#5DD44PA)
    --------------------------------------------------
    Name: HP ProBook 645 G4 Notebook PC
    rating: 0.0000
    processor: AMD® Ryzen™ APU processor
    processor_company: AMD
    processor_type: Ryzen™ APU
    generation: None
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR4-2400 SDRAM (1 x 8 GB)
    hard_disk: 1 TB 7200 rpm SATA
    graphic_card: AMD Radeon™ Vega Graphics
    display: None
    Actual_price: 79,603
    final_price: 77,246
    Discount: 2,357
    included_items: HP Overnighter Backpack (Worth ₹2,499)
    --------------------------------------------------
    **************************************************
    Name: HP ZBook 15u G5 Mobile Workstation
    rating: 0.0000
    processor: None
    processor_company: None
    processor_type: None
    generation: None
    os_installed: Windows 10 Pro 64
    ram: 16 GB DDR4-2400 SDRAM (1 x 16 GB)
    hard_disk: 512 GB SSD
    graphic_card: AMD Radeon™ Pro WX 3100 Graphics (2 GB GDDR5 dedicated)
    display: None
    Actual_price: 264,534
    final_price: 161,517
    Discount: 103,017
    included_items: HP Business Backpack (Worth ₹4,200)
    --------------------------------------------------
    Name: HP EliteBook x360 1030 G3 Notebook PC
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i7 processor
    processor_company: Intel
    processor_type: i7
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 16 GB LPDDR3-2133 SDRAM (onboard)
    hard_disk: 1 TB SSD
    graphic_card: Intel® UHD Graphics 620
    display: HP Sure View, 13.3" FHD touch screen
    Actual_price: 208,728
    final_price: 206,418
    Discount: 2,310
    included_items: HP Overnighter Backpack (Worth ₹2,499)
    --------------------------------------------------
    Name: HP Elite x2 1013 G3 Tablet
    rating: 0.0000
    processor: 8th Generation Intel® Core™ i5 processor
    processor_company: Intel
    processor_type: i5
    generation: 8
    os_installed: Windows 10 Pro 64
    ram: 8 GB LPDDR3-2133 SDRAM (onboard)
    hard_disk: 512 GB SSD
    graphic_card: Intel® UHD Graphics 620
    display: 13" Touchscreen with Corning® Gorilla® Glass 4
    Actual_price: 167,076
    final_price: 163,474
    Discount: 3,602
    included_items: HP Business Backpack (Worth ₹4,200)
    --------------------------------------------------
    Name: HP ProBook 445 G2 Notebook PC (ENERGY STAR)
    rating: 0.0000
    processor: AMD A8 APU
    processor_company: AMD
    processor_type: A8 APU
    generation: None
    os_installed: Windows 7 Professional 64
    ram: 4 GB DDR3L-1600 SDRAM (1 x 4 GB)
    hard_disk: 500 GB 5400 rpm SATA
    graphic_card: AMD Radeon™ R5
    display: None
    Actual_price: 43,990
    final_price: 43,500
    Discount: 490
    included_items: HP Overnighter Backpack (Worth ₹2,499)
    --------------------------------------------------
    Name: HP Chromebook 14 G5
    rating: 5.0000
    processor: Intel® Celeron® processor
    processor_company: Intel
    processor_type: Celeron®
    generation: None
    os_installed: Chrome OS™ 64
    ram: 8 GB LPDDR4-2400 SDRAM (onboard)
    hard_disk: 64 GB eMMC
    graphic_card: Intel® HD Graphics 500
    display: 35.56 cm(14) diagonal HD SVA eDP anti-glare LED-backlit (1366 x 768)
    Actual_price: 36,900
    final_price: 36,900
    Discount: 0
    included_items: HP Original Bag  (Worth ₹1,499) (#5DD44PA)
    --------------------------------------------------
    Name: HP Chromebook 14 G5
    rating: 0.0000
    processor: Intel® Celeron® processor
    processor_company: Intel
    processor_type: Celeron®
    generation: None
    os_installed: Chrome OS™ 64
    ram: 8 GB LPDDR4-2400 SDRAM (onboard)
    hard_disk: 32 GB eMMC
    graphic_card: Intel® HD Graphics 500
    display: 35.56 cm(14) diagonal HD SVA eDP anti-glare LED-backlit (1366 x 768)
    Actual_price: 35,654
    final_price: 35,654
    Discount: 0
    included_items: HP Original Bag  (Worth ₹1,499) (#5DD44PA)
    --------------------------------------------------
    Name: HP Chromebook 14 G5
    rating: 3.0000
    processor: Intel® Celeron® processor
    processor_company: Intel
    processor_type: Celeron®
    generation: None
    os_installed: Chrome OS™ 64
    ram: 4 GB LPDDR4-2400 SDRAM (onboard)
    hard_disk: 32 GB eMMC
    graphic_card: Intel® HD Graphics 500
    display: 35.56 cm(14) diagonal HD SVA eDP anti-glare LED-backlit (1366 x 768)
    Actual_price: 30,749
    final_price: 30,749
    Discount: 0
    included_items: HP Original Bag  (Worth ₹1,499) (#5DD44PA)
    --------------------------------------------------
    Name: HP Chromebook 14 G5
    rating: 0.0000
    processor: Intel® Celeron® processor
    processor_company: Intel
    processor_type: Celeron®
    generation: None
    os_installed: Chrome OS™ 64
    ram: 4 GB LPDDR4-2400 SDRAM (onboard)
    hard_disk: 16 GB eMMC
    graphic_card: Intel® HD Graphics 500
    display: 35.56 cm(14) diagonal HD SVA eDP anti-glare LED-backlit (1366 x 768)
    Actual_price: 30,000
    final_price: 30,000
    Discount: 0
    included_items: HP Original Bag  (Worth ₹1,499) (#5DD44PA)
    --------------------------------------------------
    Name: HP EliteBook x360 1020 G2
    rating: 0.0000
    processor: 7th Generation Intel® Core™ i7 processor
    processor_company: Intel
    processor_type: i7
    generation: 7
    os_installed: Windows 10 Pro 64
    ram: 16 GB LPDDR3-1866 SDRAM
    hard_disk: 512 GB PCIe® NVMe™ M.2 SSD
    graphic_card: Intel® HD Graphics 620
    display: None
    Actual_price: 225,000
    final_price: 225,000
    Discount: 0
    included_items: HP Business Backpack (Worth ₹4,200)
    --------------------------------------------------
    Name: HP EliteBook 1040 G4 Notebook PC
    rating: 0.0000
    processor: 7th Generation Intel® Core™ i7 processor
    processor_company: Intel
    processor_type: i7
    generation: 7
    os_installed: Windows 10 Pro 64
    ram: 16 GB DDR4-2133 SDRAM (onboard)
    hard_disk: 1 TB PCIe® NVMe™ SSD
    graphic_card: Intel® HD Graphics 630
    display: 35.56 cm(14) diagonal FHD IPS eDP LED-backlit touch screen with Corning® Gorilla® Glass 4 and privacy screen, 700 cd/m², 100% sRGB (1920 x 1080)
    Actual_price: 252,000
    final_price: 252,000
    Discount: 0
    included_items: HP Business Backpack (Worth ₹4,200)
    --------------------------------------------------
    Name: HP 250 G6 Notebook PC
    rating: 0.0000
    processor: 6th Generation Intel® Core™ i3 processor
    processor_company: Intel
    processor_type: i3
    generation: 6
    os_installed: FreeDos 2.0
    ram: 4 GB DDR4-2133 SDRAM (1 x 4 GB)
    hard_disk: 1 TB 5400 rpm SATA
    graphic_card: Intel® HD Graphics 520
    display: None
    Actual_price: 34,810
    final_price: 34,610
    Discount: 200
    included_items: HP Original Bag  (Worth ₹1,499) (#5DD44PA)
    --------------------------------------------------
    Name: HP ProBook 445 G2 Notebook PC (ENERGY STAR)
    rating: 0.0000
    processor: AMD A10 APU
    processor_company: AMD
    processor_type: A10 APU
    generation: None
    os_installed: Windows 10 Pro 64
    ram: 4 GB DDR3L-1600 SDRAM (1 x 4 GB)
    hard_disk: 500 GB 7200 rpm SATA
    graphic_card: AMD Radeon™ R7 M260DX (1 GB DDR3 dedicated, dual graphics)
    display: 14" diagonal HD anti-glare LED-backlit (1366 x 768)
    Actual_price: 55,008
    final_price: 54,300
    Discount: 708
    included_items: HP Overnighter Backpack (Worth ₹2,499)
    --------------------------------------------------
    Name: HP ProBook 445 G2 Notebook PC (ENERGY STAR)
    rating: 0.0000
    processor: AMD A10 APU
    processor_company: AMD
    processor_type: A10 APU
    generation: None
    os_installed: Windows 10 Pro 64
    ram: 8 GB DDR3L-1600 SDRAM (1 x 8 GB)
    hard_disk: 500 GB 5400 rpm SATA
    graphic_card: AMD Radeon™ R6
    display: 14" diagonal HD anti-glare LED-backlit (1366 x 768)
    Actual_price: 79,000
    final_price: 78,000
    Discount: 1,000
    included_items: HP Overnighter Backpack (Worth ₹2,499)
    --------------------------------------------------
    **************************************************
    

List of products we got after scraping


```python
prod_list
```




    [{'Name': 'HP ENVY x360 - 13-ag0035au',
      'actual_price': '83,496',
      'discout': '10,506',
      'display': '13.3" FHD multitouch-enabled edge-to-edge glass (1920 x 1080)',
      'final_price': '72,990',
      'generation': None,
      'graphic_card': 'AMD Radeon™ Vega 8 Graphics',
      'hard_disk': '256 GB SSD',
      'included_items': '',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': 'AMD Ryzen™ 5 processor',
      'processor_company': 'AMD',
      'processor_type': 'Ryzen™ 5',
      'ram': '8 GB DDR4-2400 SDRAM (onboard)',
      'rating': '3.6667'},
     {'Name': 'HP Gaming Pavilion - 15-cx0140tx',
      'actual_price': '86,476',
      'discout': '13,486',
      'display': None,
      'final_price': '72,990',
      'generation': '8',
      'graphic_card': 'NVIDIA® GeForce® GTX 1050 (4 GB GDDR5 dedicated)',
      'hard_disk': '1 TB 7200 rpm SATA',
      'included_items': 'HP Odyssey backpack (Worth ₹3,499),Microsoft Office Home and Student',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2666 SDRAM (1 x 8 GB)',
      'rating': '5.0000'},
     {'Name': 'HP Notebook - 15-da0435tx',
      'actual_price': '50,292',
      'discout': '5,712',
      'display': None,
      'final_price': '44,580',
      'generation': '7',
      'graphic_card': 'NVIDIA® GeForce® MX110 (2 GB DDR3 dedicated)',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': '',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': '7th Generation Intel® Core™ i3 processor',
      'processor_company': 'Intel',
      'processor_type': 'i3',
      'ram': '8 GB DDR4-2133 SDRAM (1 x 8 GB)',
      'rating': '4.0000'},
     {'Name': 'HP Notebook - 15g-dr0006tx',
      'actual_price': '66,137',
      'discout': '7,146',
      'display': None,
      'final_price': '58,991',
      'generation': '8',
      'graphic_card': 'NVIDIA® GeForce® MX110 (2 GB DDR3 dedicated)',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Original Laptop Bag (Worth ₹1,123),1 Year On-site Warranty,Microsoft Office Home and Student',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '4.0625'},
     {'Name': 'HP Notebook 15-da1030tu',
      'actual_price': '50,720',
      'discout': '3,730',
      'display': None,
      'final_price': '46,990',
      'generation': '8',
      'graphic_card': None,
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Original Laptop Bag (Worth ₹1,123),Microsoft Office Home and Student',
      'os_installed': 'Windows 10 Home 64',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '4 GB DDR4-2400 SDRAM',
      'rating': '2.0000'},
     {'Name': 'HP Pavilion - 15-cw0027au',
      'actual_price': '70,025',
      'discout': '10,035',
      'display': None,
      'final_price': '59,990',
      'generation': None,
      'graphic_card': 'AMD Radeon™ Vega 8 Graphics',
      'hard_disk': '1 TB 5400 rpm SATA + 128 GB SSD',
      'included_items': 'HP Trendsetter Bag (Worth ₹2,356)',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': 'AMD Ryzen™ 5 processor',
      'processor_company': 'AMD',
      'processor_type': 'Ryzen™ 5',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '3.0000'},
     {'Name': 'HP Pavilion 13-an0045tu',
      'actual_price': '75,379',
      'discout': '2,503',
      'display': None,
      'final_price': '72,876',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '128 GB M.2 SSD',
      'included_items': 'HP Original Laptop Bag (Worth ₹1,123)',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2400 SDRAM (onboard)',
      'rating': '0.0000'},
     {'Name': 'HP Pavilion 13-an0046tu',
      'actual_price': '75,875',
      'discout': '0',
      'display': None,
      'final_price': '75,875',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '256 GB PCIe® NVMe™ M.2 SSD',
      'included_items': 'HP Original Laptop Bag (Worth ₹1,123)',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2400 SDRAM (onboard)',
      'rating': '0.0000'},
     {'Name': 'HP Pavilion 14-ce1000tu',
      'actual_price': '69,490',
      'discout': '9,500',
      'display': None,
      'final_price': '59,990',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '256 GB SSD',
      'included_items': 'HP Trendsetter Bag (Worth ₹2,356),Microsoft Office Home and Student',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '3.0000'},
     {'Name': 'HP Pavilion 14-ce1000tx',
      'actual_price': '78,590',
      'discout': '6,600',
      'display': '14" diagonal FHD IPS BrightView WLED-backlit (1920 x 1080)',
      'final_price': '71,990',
      'generation': '8',
      'graphic_card': 'NVIDIA® GeForce® MX150 (2 GB GDDR5 dedicated)',
      'hard_disk': '256 GB PCIe® NVMe™ M.2 SSD',
      'included_items': 'HP Trendsetter Bag (Worth ₹2,356),Microsoft Office Home and Student',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '4.5000'},
     {'Name': 'HP Pavilion 14-ce1001tx',
      'actual_price': '80,946',
      'discout': '10,957',
      'display': None,
      'final_price': '69,989',
      'generation': '8',
      'graphic_card': 'NVIDIA® GeForce® MX150 (2 GB GDDR5 dedicated)',
      'hard_disk': '1 TB 5400 rpm SATA + 128 GB SSD',
      'included_items': 'HP Trendsetter Bag (Worth ₹2,356),Microsoft Office Home and Student',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '4.2857'},
     {'Name': 'HP Pavilion 14-ce1003tx',
      'actual_price': '108,460',
      'discout': '4,685',
      'display': None,
      'final_price': '103,775',
      'generation': '8',
      'graphic_card': 'NVIDIA® GeForce® MX150 (2 GB GDDR5 dedicated)',
      'hard_disk': '512 GB SSD',
      'included_items': 'HP Trendsetter Bag (Worth ₹2,356),Microsoft Office Home and Student',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': '8th Generation Intel® Core™ i7 processor',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '16 GB DDR4-2400 SDRAM (1 x 16 GB)',
      'rating': '4.0000'},
     {'Name': 'HP Pavilion 15-cs1000tx',
      'actual_price': '70,668',
      'discout': '7,678',
      'display': None,
      'final_price': '62,990',
      'generation': '8',
      'graphic_card': 'NVIDIA® GeForce® MX130 (2 GB GDDR5 dedicated)',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Trendsetter Bag (Worth ₹2,356),1 Year On-site Warranty,Microsoft Office Home and Student',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '2.1071'},
     {'Name': 'HP Pavilion 15-cs1052tx',
      'actual_price': '88,547',
      'discout': '6,557',
      'display': None,
      'final_price': '81,990',
      'generation': '8',
      'graphic_card': 'NVIDIA® GeForce® MX150 (4 GB GDDR5 dedicated)',
      'hard_disk': '2 TB 5400 rpm SATA',
      'included_items': 'HP Original Laptop Bag (Worth ₹1,123),Microsoft Office Home and Student',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': '8th Generation Intel® Core™ i7 processor',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP Pavilion x360 - 11-ad106tu',
      'actual_price': '50,005',
      'discout': '4,425',
      'display': None,
      'final_price': '45,580',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Active Pen (Worth ₹3,145),HP Trendsetter Bag (Worth ₹2,356)',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': '8th Generation Intel® Core™ i3 processor',
      'processor_company': 'Intel',
      'processor_type': 'i3',
      'ram': '4 GB DDR4-2400 SDRAM (1 x 4 GB)',
      'rating': '4.0000'},
     {'Name': 'HP Pavilion x360 - 14-cd0050tx',
      'actual_price': '64,887',
      'discout': '4,307',
      'display': None,
      'final_price': '60,580',
      'generation': '8',
      'graphic_card': '2 GB NVIDIA GeForce MX 130 Graphics',
      'hard_disk': '1 TB 5400 rpm SATA SSHD',
      'included_items': 'HP Original Laptop Bag (Worth ₹1,123),HP Active Pen (Worth ₹3,145),Microsoft Office Home and Student',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': '8th Generation Intel® Core™ i3 processor',
      'processor_company': 'Intel',
      'processor_type': 'i3',
      'ram': '4 GB DDR4-2400 SDRAM (1 x 4 GB), 2 SODIMM slots (upgradeable to 12 GB)',
      'rating': '4.2500'},
     {'Name': 'HP Pavilion x360 - 14-cd0053tx',
      'actual_price': '86,834',
      'discout': '5,959',
      'display': None,
      'final_price': '80,875',
      'generation': '8',
      'graphic_card': '2 GB NVIDIA GeForce MX 130 Graphics',
      'hard_disk': '1TB 5400 RPM SATA+ 16 GB SSD Intel Optan',
      'included_items': 'HP Original Laptop Bag (Worth ₹1,123),HP Active Pen (Worth ₹3,145),Microsoft Office Home and Student',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB), 2 SODIMM Slots (upgradeable to 12 GB)',
      'rating': '4.3333'},
     {'Name': 'HP Pavilion x360 - 14-cd0078tu',
      'actual_price': '61,247',
      'discout': '7,667',
      'display': None,
      'final_price': '53,580',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '256 GB SSD',
      'included_items': 'HP Original Laptop Bag (Worth ₹1,123),HP Active Pen (Worth ₹3,145),Microsoft Office Home and Student',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': '8th Generation Intel® Core™ i3 processor',
      'processor_company': 'Intel',
      'processor_type': 'i3',
      'ram': '4 GB DDR4-2400 SDRAM (1 x 4 GB)',
      'rating': '3.1429'},
     {'Name': 'HP Pavilion x360 - 14-cd0080tu',
      'actual_price': '77,841',
      'discout': '3,966',
      'display': None,
      'final_price': '73,875',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '1 TB 5400 rpm SATA SSHD (8 GB NAND)',
      'included_items': 'HP Original Laptop Bag (Worth ₹1,123),HP Active Pen (Worth ₹3,145),Microsoft Office Home and Student',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB), 2 SODIMM Slots (upgradeable to 12 GB)',
      'rating': '2.0000'},
     {'Name': 'HP Pavilion x360 - 14-cd0081tu',
      'actual_price': '82,659',
      'discout': '11,783',
      'display': None,
      'final_price': '70,876',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '256 GB SSD',
      'included_items': 'HP Original Laptop Bag (Worth ₹1,123),HP Active Pen (Worth ₹3,145),Microsoft Office Home and Student',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ProBook 440 G6 Notebook PC',
      'actual_price': '79,347',
      'discout': '2,348',
      'display': None,
      'final_price': '76,999',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Overnighter Backpack (Worth ₹2,499)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP Notebook - 14s-cf0055tu',
      'actual_price': '39,000',
      'discout': '920',
      'display': '14" diagonal FHD IPS BrightView micro-edge WLED-backlit (1920 x 1080)',
      'final_price': '38,080',
      'generation': '7',
      'graphic_card': 'Intel® HD Graphics 620',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Original Laptop Bag (Worth ₹1,123),Microsoft Office Home and Student',
      'os_installed': 'Windows 10 Home 64',
      'processor': '7th Generation Intel® Core™ i3 processor',
      'processor_company': 'Intel',
      'processor_type': 'i3',
      'ram': '4 GB DDR4-2133 SDRAM (1 x 4 GB)',
      'rating': '3.3333'},
     {'Name': 'HP Notebook - 15-db0209au',
      'actual_price': '26,000',
      'discout': '1,150',
      'display': '15.6" diagonal HD SVA BrightView micro-edge WLED-backlit (1366 x 768)',
      'final_price': '24,850',
      'generation': None,
      'graphic_card': 'AMD Radeon™ R3 Graphics',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Original Laptop Bag (Worth ₹1,123),1 Year On-site Warranty',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': 'AMD Dual-Core A-Series processor',
      'processor_company': 'AMD',
      'processor_type': 'Dual-Core A-Series',
      'ram': '4 GB DDR4-1866 SDRAM (1 x 4 GB)',
      'rating': '5.0000'},
     {'Name': 'HP ProBook x360 440 G1 Notebook PC',
      'actual_price': '89,587',
      'discout': '2,652',
      'display': '14" FHD Touchscreen Display',
      'final_price': '86,935',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '256 GB SSD',
      'included_items': 'HP Active Pen (Worth ₹3,145),HP Overnighter Backpack (Worth ₹2,499)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP EliteBook x360 1030 G3 Notebook PC',
      'actual_price': '176,627',
      'discout': '1,543',
      'display': None,
      'final_price': '175,084',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '512 GB SSD',
      'included_items': 'HP Active Pen (Worth ₹3,145),HP Original Bag  (Worth ₹1,499) (#5DD44PA)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i7 processor',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '8 GB DDR4-2133 SDRAM (onboard)',
      'rating': '0.0000'},
     {'Name': 'HP Notebook - 15-db0186au',
      'actual_price': '38,622',
      'discout': '5,542',
      'display': None,
      'final_price': '33,080',
      'generation': None,
      'graphic_card': 'AMD Radeon™ Vega 3 Graphics',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Original Laptop Bag (Worth ₹1,123),Microsoft Office Home and Student',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': 'AMD Ryzen™ 3 processor',
      'processor_company': 'AMD',
      'processor_type': 'Ryzen™ 3',
      'ram': '4 GB DDR4-2400 SDRAM (1 x 4 GB)',
      'rating': '3.6667'},
     {'Name': 'HP ProBook 440 G5 Notebook PC',
      'actual_price': '77,427',
      'discout': '5,025',
      'display': None,
      'final_price': '72,402',
      'generation': '7',
      'graphic_card': 'Intel® HD Graphics 620',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Overnighter Backpack (Worth ₹2,499)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '7th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '4 GB DDR4-2400 SDRAM (1 x 4 GB)',
      'rating': '0.0000'},
     {'Name': 'HP Notebook - 15-da0327tu',
      'actual_price': '36,481',
      'discout': '1,651',
      'display': None,
      'final_price': '34,830',
      'generation': '7',
      'graphic_card': 'Intel® HD Graphics 620',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Original Laptop Bag (Worth ₹1,123),Microsoft Office Home and Student',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': '7th Generation Intel® Core™ i3 processor',
      'processor_company': 'Intel',
      'processor_type': 'i3',
      'ram': '4 GB DDR4-2133 SDRAM (1 x 4 GB)',
      'rating': '2.3636'},
     {'Name': 'HP Notebook - 15-da0295tu',
      'actual_price': '27,381',
      'discout': '1,269',
      'display': None,
      'final_price': '26,112',
      'generation': '7',
      'graphic_card': 'Intel® UHD Graphics 605',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Original Laptop Bag (Worth ₹1,123)',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': 'Intel® Pentium® Silver processor',
      'processor_company': 'Intel',
      'processor_type': 'Pentium® Silver',
      'ram': '4 GB DDR4-2400 SDRAM (1 x 4 GB)',
      'rating': '2.0000'},
     {'Name': 'HP Notebook - 15-da0326tu',
      'actual_price': '35,962',
      'discout': '2,132',
      'display': None,
      'final_price': '33,830',
      'generation': '7',
      'graphic_card': 'Intel® HD Graphics 620',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Original Laptop Bag (Worth ₹1,123)',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': '7th Generation Intel® Core™ i3 processor',
      'processor_company': 'Intel',
      'processor_type': 'i3',
      'ram': '4 GB DDR4-2133 SDRAM (1 x 4 GB)',
      'rating': '3.0000'},
     {'Name': 'HP EliteBook 830 G5 Notebook PC',
      'actual_price': '122,227',
      'discout': '1,068',
      'display': None,
      'final_price': '121,159',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '512 GB SSD',
      'included_items': 'HP Business Backpack (Worth ₹4,200)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP EliteBook 840r G4 Notebook PC',
      'actual_price': '112,883',
      'discout': '986',
      'display': None,
      'final_price': '111,897',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '1 TB 7200 rpm SATA+128 GB SSD',
      'included_items': 'HP Business Backpack (Worth ₹4,200)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP EliteBook 840r G4 Notebook PC',
      'actual_price': '115,571',
      'discout': '1,010',
      'display': None,
      'final_price': '114,561',
      'generation': '8',
      'graphic_card': 'Intel® UHD graphics 620',
      'hard_disk': '1 TB 7200 rpm SATA',
      'included_items': 'HP Business Backpack (Worth ₹4,200)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i7 processor',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ProBook x360 440 G1 Notebook PC',
      'actual_price': '80,627',
      'discout': '2,387',
      'display': '14" FHD Touchscreen',
      'final_price': '78,240',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '256 GB SSD',
      'included_items': 'HP Overnighter Backpack (Worth ₹2,499)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i3 processor',
      'processor_company': 'Intel',
      'processor_type': 'i3',
      'ram': '4 GB DDR4-2400 SDRAM (1 x 4 GB)',
      'rating': '0.0000'},
     {'Name': 'HP EliteBook 840r G4 Notebook PC',
      'actual_price': '127,987',
      'discout': '1,118',
      'display': None,
      'final_price': '126,869',
      'generation': '8',
      'graphic_card': 'Intel® UHD graphics 620',
      'hard_disk': '1 TB 7200 rpm SATA+128 GB SSD',
      'included_items': 'HP Business Backpack (Worth ₹4,200)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i7 processor',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '16 GB DDR4-2400 SDRAM (1 x 16 GB)',
      'rating': '0.0000'},
     {'Name': 'HP EliteBook 840r G4 Notebook PC',
      'actual_price': '102,643',
      'discout': '896',
      'display': None,
      'final_price': '101,747',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '1 TB 7200 rpm SATA',
      'included_items': 'HP Business Backpack (Worth ₹4,200)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ProBook x360 440 G1 Notebook PC',
      'actual_price': '102,643',
      'discout': '3,038',
      'display': '14" FHD Touchscreen',
      'final_price': '99,605',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '512 GB SSD',
      'included_items': 'HP Active Pen (Worth ₹3,145),HP Overnighter Backpack (Worth ₹2,499)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '2.0000'},
     {'Name': 'HP Pavilion - 15-bc407tx',
      'actual_price': '89,474',
      'discout': '14,533',
      'display': None,
      'final_price': '74,941',
      'generation': '8',
      'graphic_card': 'NVIDIA® GeForce® GTX 1050 (4 GB GDDR5 dedicated)',
      'hard_disk': '1 TB 5400 rpm SATA + 128 GB SSD',
      'included_items': 'HP Trendsetter Bag (Worth ₹2,356),1 Year On-site Warranty',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '5.0000'},
     {'Name': 'HP Pavilion - 15-bc406tx',
      'actual_price': '80,481',
      'discout': '10,751',
      'display': None,
      'final_price': '69,730',
      'generation': '8',
      'graphic_card': 'NVIDIA® GeForce® GTX 1050 (4 GB GDDR5 dedicated)',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Trendsetter Bag (Worth ₹2,356),1 Year On-site Warranty',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '3.1667'},
     {'Name': 'HP Pavilion x360 - 11-ad105tu',
      'actual_price': '39,942',
      'discout': '2,361',
      'display': None,
      'final_price': '37,581',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 605',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Trendsetter Bag (Worth ₹2,356)',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': 'Intel® Pentium® processor',
      'processor_company': 'Intel',
      'processor_type': 'Pentium®',
      'ram': '4 GB DDR4-2400 SDRAM (1 x 4 GB)',
      'rating': '2.0000'},
     {'Name': 'HP ZBook 15v G5 Mobile Workstation',
      'actual_price': '225,937',
      'discout': '22,095',
      'display': None,
      'final_price': '203,842',
      'generation': None,
      'graphic_card': 'NVIDIA® Quadro® P600 (4 GB GDDR5 dedicated)',
      'hard_disk': '1 TB 7200 rpm SATA',
      'included_items': 'HP Original Bag  (Worth ₹1,499) (#5DD44PA),3 Years Warranty',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '16 GB DDR4-2666 SDRAM (1 x 16 GB)',
      'processor_company': None,
      'processor_type': None,
      'ram': None,
      'rating': ''},
     {'Name': 'HP ZBook 15v G5 Mobile Workstation',
      'actual_price': '233,611',
      'discout': '22,861',
      'display': None,
      'final_price': '210,750',
      'generation': None,
      'graphic_card': 'NVIDIA® Quadro® P600 (4 GB GDDR5 dedicated)',
      'hard_disk': '256 GB SSD',
      'included_items': 'HP Original Bag  (Worth ₹1,499) (#5DD44PA),3 Years Warranty',
      'os_installed': 'Windows 10 Pro 64',
      'processor': None,
      'processor_company': None,
      'processor_type': None,
      'ram': '16 GB DDR4-2666 SDRAM (1 x 16 GB)',
      'rating': ''},
     {'Name': 'HP ZBook 15v G5 Mobile Workstation',
      'actual_price': '216,245',
      'discout': '21,125',
      'display': None,
      'final_price': '195,120',
      'generation': None,
      'graphic_card': 'NVIDIA® Quadro® P600 (4 GB GDDR5 dedicated)',
      'hard_disk': '1 TB 7200 rpm SATA',
      'included_items': 'HP Original Bag  (Worth ₹1,499) (#5DD44PA),3 Years Warranty',
      'os_installed': 'Windows 10 Pro 64',
      'processor': None,
      'processor_company': None,
      'processor_type': None,
      'ram': '16 GB DDR4-2666 SDRAM (1 x 16 GB)',
      'rating': ''},
     {'Name': 'HP ZBook 15v G5 Mobile Workstation',
      'actual_price': '203,281',
      'discout': '19,828',
      'display': None,
      'final_price': '183,453',
      'generation': None,
      'graphic_card': 'NVIDIA® Quadro® P600 (4 GB GDDR5 dedicated)',
      'hard_disk': '2 TB 5400 rpm SATA',
      'included_items': 'HP Original Bag  (Worth ₹1,499) (#5DD44PA),3 Years Warranty',
      'os_installed': 'Windows 10 Pro 64',
      'processor': None,
      'processor_company': None,
      'processor_type': None,
      'ram': '16 GB DDR4-2666 SDRAM (1 x 16 GB)',
      'rating': ''},
     {'Name': 'HP ZBook 15v G5 Mobile Workstation',
      'actual_price': '181,301',
      'discout': '17,100',
      'display': None,
      'final_price': '164,201',
      'generation': None,
      'graphic_card': 'NVIDIA® Quadro® P600 (4 GB GDDR5 dedicated)',
      'hard_disk': '1 TB 7200 rpm SATA',
      'included_items': 'HP Original Bag  (Worth ₹1,499) (#5DD44PA),3 Years Warranty',
      'os_installed': 'Windows 10 Pro 64',
      'processor': None,
      'processor_company': None,
      'processor_type': None,
      'ram': '16 GB DDR4-2666 SDRAM (1 x 16 GB)',
      'rating': ''},
     {'Name': 'OMEN by HP - 15-dc0084tx',
      'actual_price': '134,859',
      'discout': '9,422',
      'display': None,
      'final_price': '125,437',
      'generation': '8',
      'graphic_card': 'NVIDIA® GeForce® GTX 1050 Ti (4 GB GDDR5 dedicated)',
      'hard_disk': '1 TB 7200 rpm SATA + 128 GB SSD',
      'included_items': 'Omen Gaming Bag (Worth ₹4,950)',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': '8th Generation Intel® Core™ i7 processor',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '16 GB DDR4-2666 SDRAM (1 x 16 GB)',
      'rating': '4.3333'},
     {'Name': 'HP Pavilion x360 - 14-cd0051tx',
      'actual_price': '82,659',
      'discout': '6,845',
      'display': None,
      'final_price': '75,814',
      'generation': '8',
      'graphic_card': 'NVIDIA® GeForce® MX130 (2 GB GDDR5 dedicated)',
      'hard_disk': '1 TB 5400 rpm SATA SSHD',
      'included_items': 'HP Original Laptop Bag (Worth ₹1,123),HP Active Pen (Worth ₹3,145),Microsoft Office Home and Student',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '4.0000'},
     {'Name': 'HP 348 G4 Notebook PC',
      'actual_price': '74,867',
      'discout': '3,714',
      'display': None,
      'final_price': '71,153',
      'generation': '7',
      'graphic_card': 'Intel® HD Graphics 620',
      'hard_disk': '1 TB 7200 rpm SATA',
      'included_items': 'HP Original Bag  (Worth ₹1,499) (#5DD44PA)',
      'os_installed': 'Windows 10 Pro 64 – HP recommends Windows 10 Pro.',
      'processor': '7th Generation Intel® Core™ i7 processor',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '8 GB DDR4-2133 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP 348 G4 Notebook PC',
      'actual_price': '55,539',
      'discout': '2,756',
      'display': '14" diagonal HD SVA eDP anti-glare LED-backlit (1366 x 768)',
      'final_price': '52,783',
      'generation': '7',
      'graphic_card': 'Intel® HD Graphics 620',
      'hard_disk': '1 TB 7200 rpm SATA',
      'included_items': 'HP Original Bag  (Worth ₹1,499) (#5DD44PA)',
      'os_installed': 'FreeDOS',
      'processor': '7th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2133 SDRAM (1 x 8 GB)',
      'rating': '5.0000'},
     {'Name': 'HP 348 G4 Notebook PC',
      'actual_price': '65,907',
      'discout': '3,269',
      'display': '14" diagonal HD SVA eDP anti-glare LED-backlit (1366 x 768)',
      'final_price': '62,638',
      'generation': '7',
      'graphic_card': 'Intel® HD Graphics 620',
      'hard_disk': '1 TB 7200 rpm SATA',
      'included_items': 'HP Original Bag  (Worth ₹1,499) (#5DD44PA)',
      'os_installed': 'Windows 10 Pro 64 – HP recommends Windows 10 Pro.',
      'processor': '7th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2133 SDRAM (1 x 8 GB)',
      'rating': '4.0000'},
     {'Name': 'HP EliteBook 830 G5 Notebook PC',
      'actual_price': '138,867',
      'discout': '1,213',
      'display': 'HP Sure View Integrated Privacy Screen 33.78 cm(13.3) diagonal FHD IPS anti-glare LED-backlit, 300 cd/m², 72% sRGB (1920 x 1080)',
      'final_price': '137,654',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '512 GB PCIe® NVMe™ M.2 SSD',
      'included_items': 'HP Business Backpack (Worth ₹4,200)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i7 processor',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '5.0000'},
     {'Name': 'HP EliteBook x360 1030 G2',
      'actual_price': '138,867',
      'discout': '1,213',
      'display': None,
      'final_price': '137,654',
      'generation': '7',
      'graphic_card': 'Intel® HD Graphics 620',
      'hard_disk': '256 GB PCIe® NVMe™ SSD',
      'included_items': 'HP Business Backpack (Worth ₹4,200)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '7th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB LPDDR3-1866 SDRAM (onboard)',
      'rating': '0.0000'},
     {'Name': 'HP EliteBook 1040 G4 Notebook PC',
      'actual_price': '236,787',
      'discout': '2,703',
      'display': '35.56 cm(14) diagonal FHD IPS eDP anti-glare LED-backlit, 300 cd/m², 72% sRGB with privacy screen (1920 x 1080)',
      'final_price': '234,084',
      'generation': '7',
      'graphic_card': 'Intel® HD Graphics 630',
      'hard_disk': '1 TB PCIe® NVMe™ SSD',
      'included_items': 'HP Business Backpack (Worth ₹4,200)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '7th Generation Intel® Core™ i7 processor',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '16 GB DDR4-2133 SDRAM (onboard)',
      'rating': '0.0000'},
     {'Name': 'HP EliteBook x360 1030 G2 (ENERGY STAR)',
      'actual_price': '161,267',
      'discout': '1,409',
      'display': None,
      'final_price': '159,858',
      'generation': '7',
      'graphic_card': 'Intel® HD Graphics 620',
      'hard_disk': '256 GB M.2 SATA TLC SSD',
      'included_items': 'HP Business Backpack (Worth ₹4,200)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '7th Generation Intel® Core™ i7 processor',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '8 GB DDR4-2133 SDRAM (onboard)',
      'rating': '5.0000'},
     {'Name': 'HP ProBook 450 G6 Notebook PC',
      'actual_price': '80,627',
      'discout': '2,387',
      'display': None,
      'final_price': '78,240',
      'generation': '8',
      'graphic_card': 'NVIDIA® GeForce® MX130 (2 GB DDR5 dedicated)',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Overnighter Backpack (Worth ₹2,499)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i5 processor (i5-8265U)',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ProBook 440 G6 Notebook PC',
      'actual_price': '65,267',
      'discout': '1,932',
      'display': None,
      'final_price': '63,335',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Overnighter Backpack (Worth ₹2,499)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i3 processor',
      'processor_company': 'Intel',
      'processor_type': 'i3',
      'ram': '4 GB DDR4-2400 SDRAM (1 x 4 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ProBook 440 G6 Notebook PC',
      'actual_price': '67,827',
      'discout': '2,008',
      'display': None,
      'final_price': '65,819',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Overnighter Backpack (Worth ₹2,499)',
      'os_installed': 'FreeDOS 3.0',
      'processor': '8th Generation Intel® Core™ i5 processor (i5-8265U)',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ProBook 440 G6 Notebook PC',
      'actual_price': '92,147',
      'discout': '2,728',
      'display': None,
      'final_price': '89,419',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Overnighter Backpack (Worth ₹2,499)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i7 processor',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ProBook 430 G6 Notebook PC',
      'actual_price': '79,987',
      'discout': '2,368',
      'display': None,
      'final_price': '77,619',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Overnighter Backpack (Worth ₹2,499)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i5 processor (i5-8265U)',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ProBook 430 G6 Notebook PC',
      'actual_price': '89,587',
      'discout': '2,652',
      'display': None,
      'final_price': '86,935',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Overnighter Backpack (Worth ₹2,499)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i7 processor',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ProBook 430 G6 Notebook PC',
      'actual_price': '106,227',
      'discout': '3,145',
      'display': None,
      'final_price': '103,082',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Overnighter Backpack (Worth ₹2,499)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i7 processor (i7-8565U)',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '16 GB DDR4-2400 SDRAM (1 x 16 GB)',
      'rating': '0.0000'},
     {'Name': 'HP Pavilion x360 14-dh0101tu',
      'actual_price': '51,826',
      'discout': '1,836',
      'display': None,
      'final_price': '49,990',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '256 GB PCIe® NVMe™ M.2 SSD',
      'included_items': 'HP Active Pen (Worth ₹3,145)',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': '8th Generation Intel® Core™ i3 processor',
      'processor_company': 'Intel',
      'processor_type': 'i3',
      'ram': '4 GB DDR4-2400 SDRAM (1 x 4 GB)',
      'rating': '0.0000'},
     {'Name': 'HP Pavilion x360 14-dh0047tu',
      'actual_price': '57,179',
      'discout': '2,189',
      'display': None,
      'final_price': '54,990',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Active Pen (Worth ₹3,145)',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': '8th Generation Intel® Core™ i3 processor',
      'processor_company': 'Intel',
      'processor_type': 'i3',
      'ram': '4 GB DDR4-2400 SDRAM (1 x 4 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ZBook 15u G5 Mobile Workstation',
      'actual_price': '127,636',
      'discout': '0',
      'display': None,
      'final_price': '127,636',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '512 GB PCIe® NVMe™ SSD',
      'included_items': '',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP Pavilion x360 14-dh0045tx',
      'actual_price': '101,073',
      'discout': '5,083',
      'display': None,
      'final_price': '95,990',
      'generation': '8',
      'graphic_card': 'NVIDIA® GeForce® MX250 (2 GB GDDR5 dedicated)',
      'hard_disk': '512 GB PCIe® NVMe™ M.2 SSD',
      'included_items': 'HP Active Pen (Worth ₹3,145)',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': '8th Generation Intel® Core™ i7 processor',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '16 GB DDR4-2400 SDRAM (2 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP Pavilion x360 14-dh0043tx',
      'actual_price': '80,732',
      'discout': '742',
      'display': None,
      'final_price': '79,990',
      'generation': '8',
      'graphic_card': 'NVIDIA® GeForce® MX130 (2 GB GDDR5 dedicated)',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Active Pen (Worth ₹3,145)',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP Pavilion x360 14-dh0042tu',
      'actual_price': '77,520',
      'discout': '3,530',
      'display': None,
      'final_price': '73,990',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Active Pen (Worth ₹3,145)',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP Pavilion x360 14-dh0043tu',
      'actual_price': '71,097',
      'discout': '3,107',
      'display': None,
      'final_price': '67,990',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '256 GB PCIe® NVMe™ M.2 SSD',
      'included_items': 'HP Active Pen (Worth ₹3,145)',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP Pavilion x360 14-dh0044tx',
      'actual_price': '62,532',
      'discout': '2,542',
      'display': '14" diagonal FHD IPS anti-glare micro-edge WLED-backlit touch screen (1920 x 1080)',
      'final_price': '59,990',
      'generation': '8',
      'graphic_card': 'NVIDIA® GeForce® MX130 (2 GB GDDR5 dedicated)',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Active Pen (Worth ₹3,145)',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': '8th Generation Intel® Core™ i3 processor',
      'processor_company': 'Intel',
      'processor_type': 'i3',
      'ram': '4 GB DDR4-2400 SDRAM (1 x 4 GB)',
      'rating': '0.0000'},
     {'Name': 'HP Notebook - 15-da0073tx',
      'actual_price': '40,458',
      'discout': '4,951',
      'display': '39.62 cm(15.6) diagonal FHD SVA anti-glare micro-edge WLED-backlit (1920 x 1080)',
      'final_price': '35,507',
      'generation': '7',
      'graphic_card': 'NVIDIA® GeForce® MX110 (2 GB DDR3 dedicated)',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Original Laptop Bag (Worth ₹1,123),1 Year On-site Warranty',
      'os_installed': 'FreeDOS 2.0',
      'processor': '7th Generation Intel® Core™ i3 processor',
      'processor_company': 'Intel',
      'processor_type': 'i3',
      'ram': '4 GB DDR4-2133 SDRAM (1 x 4 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ZBook 14u G5 Mobile Workstation',
      'actual_price': '109,386',
      'discout': '0',
      'display': None,
      'final_price': '109,386',
      'generation': '8',
      'graphic_card': 'AMD Radeon™ Pro WX 3100 Graphics (2 GB GDDR5 dedicated)',
      'hard_disk': '512 GB HP Z Turbo Drive PCIe® SSD',
      'included_items': '',
      'os_installed': 'Windows 10 Pro',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP Notebook - 15-da0447tx',
      'actual_price': '42,369',
      'discout': '1,258',
      'display': '15.6" diagonal FHD SVA anti-glare WLED-backlit (1920 x 1080)',
      'final_price': '41,111',
      'generation': '7',
      'graphic_card': 'NVIDIA® GeForce® MX110 (2 GB GDDR5 dedicated)',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Original Laptop Bag (Worth ₹1,123)',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': '7th Generation Intel® Core™ i3 processor',
      'processor_company': 'Intel',
      'processor_type': 'i3',
      'ram': '4 GB DDR4-2133 SDRAM (1 x 4 GB)',
      'rating': '4.0000'},
     {'Name': 'HP 240 G6 Notebook PC',
      'actual_price': '35,630',
      'discout': '1,580',
      'display': None,
      'final_price': '34,050',
      'generation': '7',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Original Bag  (Worth ₹1,499) (#5DD44PA)',
      'os_installed': 'Windows 10 Home Single Language 64 – HP recommends Windows 10 Pro.',
      'processor': '7th Generation Intel® Core™ i3 processor',
      'processor_company': 'Intel',
      'processor_type': 'i3',
      'ram': '4 GB DDR4-2133 SDRAM (1 x 4 GB)',
      'rating': '2.0000'},
     {'Name': 'HP 250 G6 Notebook PC',
      'actual_price': '36,686',
      'discout': '1,653',
      'display': None,
      'final_price': '35,033',
      'generation': '7',
      'graphic_card': 'Intel® HD Graphics 620',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Original Bag  (Worth ₹1,499) (#5DD44PA)',
      'os_installed': 'Windows 10 Home Single Language 64 – HP recommends Windows 10 Pro.',
      'processor': '7th Generation Intel® Core™ i3 processor',
      'processor_company': 'Intel',
      'processor_type': 'i3',
      'ram': '4 GB DDR4-2133 SDRAM (1 x 4 GB)',
      'rating': '0.0000'},
     {'Name': 'HP Spectre x360 - 13-ap0122tu',
      'actual_price': '176,639',
      'discout': '5,764',
      'display': 'HP Sure View Integrated Privacy Screen 13.3" diagonal FHD IPS anti-glare micro-edge WLED-backlit touch screen (1920 x 1080)',
      'final_price': '170,875',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '512 GB PCIe® NVMe™ M.2 SSD',
      'included_items': '',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i7 processor',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '16 GB DDR4-2400 SDRAM (onboard)',
      'rating': '0.0000'},
     {'Name': 'HP Spectre x360 - 13-ap0121tu',
      'actual_price': '144,521',
      'discout': '3,646',
      'display': 'HP Sure View Integrated Privacy Screen 13.3" diagonal FHD IPS anti-glare micro-edge WLED-backlit touch screen (1920 x 1080)',
      'final_price': '140,875',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '256 GB PCIe® NVMe™ M.2 SSD',
      'included_items': '',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2400 SDRAM (onboard)',
      'rating': '0.0000'},
     {'Name': 'HP Spectre x360 - 13-ap0102tu',
      'actual_price': '203,404',
      'discout': '12,530',
      'display': None,
      'final_price': '190,874',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '1 TB PCIe® NVMe™ M.2 SSD',
      'included_items': 'Microsoft Office Home and Student',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': '8th Generation Intel® Core™ i7 processor',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '16 GB DDR4-2400 SDRAM (onboard)',
      'rating': '5.0000'},
     {'Name': 'HP Spectre x360 - 13-ap0101tu',
      'actual_price': '171,286',
      'discout': '10,411',
      'display': None,
      'final_price': '160,875',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '512 GB PCIe® NVMe™ M.2 SSD',
      'included_items': 'Microsoft Office Home and Student',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': '8th Generation Intel® Core™ i7 processor',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '16 GB DDR4-2400 SDRAM (onboard)',
      'rating': '2.0000'},
     {'Name': 'HP Spectre x360 - 13-ap0100tu',
      'actual_price': '139,168',
      'discout': '8,294',
      'display': None,
      'final_price': '130,874',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '256 GB PCIe® NVMe™ M.2 SSD',
      'included_items': 'Microsoft Office Home and Student',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2400 SDRAM (onboard)',
      'rating': '0.0000'},
     {'Name': 'HP ZBook 14u G5 Mobile Workstation',
      'actual_price': '160,151',
      'discout': '32,384',
      'display': None,
      'final_price': '127,767',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '512 GB HP Z Turbo Drive PCIe® SSD',
      'included_items': '',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i7 processor (i7-8650U, i7-8550U)',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP 240 G7 Notebook PC',
      'actual_price': '37,107',
      'discout': '1,840',
      'display': None,
      'final_price': '35,267',
      'generation': '7',
      'graphic_card': 'Intel® HD Graphics 620',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Original Bag  (Worth ₹1,499) (#5DD44PA)',
      'os_installed': 'FreeDOS 2.0',
      'processor': '7th Generation Intel® Core™ i3 processor',
      'processor_company': 'Intel',
      'processor_type': 'i3',
      'ram': '4 GB DDR4-2133 SDRAM (1 x 4 GB)',
      'rating': '0.0000'},
     {'Name': 'HP 240 G7 Notebook PC',
      'actual_price': '50,291',
      'discout': '2,495',
      'display': None,
      'final_price': '47,796',
      'generation': '7',
      'graphic_card': 'Intel® HD Graphics 620',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Original Bag  (Worth ₹1,499) (#5DD44PA)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '7th Generation Intel® Core™ i3 processor (i3-7020U)',
      'processor_company': 'Intel',
      'processor_type': 'i3',
      'ram': '4 GB DDR4-2133 SDRAM (1 x 4 GB)',
      'rating': '0.0000'},
     {'Name': 'HP 240 G7 Notebook PC',
      'actual_price': '49,907',
      'discout': '2,476',
      'display': None,
      'final_price': '47,431',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Original Bag  (Worth ₹1,499) (#5DD44PA)',
      'os_installed': 'FreeDOS 2.0',
      'processor': '8th Generation Intel® Core™ i5 processor (i5-8250U)',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '4 GB DDR4-2400 SDRAM (1 x 4 GB)',
      'rating': '0.0000'},
     {'Name': 'HP 240 G7 Notebook PC',
      'actual_price': '63,987',
      'discout': '3,175',
      'display': None,
      'final_price': '60,812',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Original Bag  (Worth ₹1,499) (#5DD44PA)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i5 processor (i5-8250U)',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP 348 G4 Notebook PC (ENERGY STAR)',
      'actual_price': '51,827',
      'discout': '2,571',
      'display': None,
      'final_price': '49,256',
      'generation': '7',
      'graphic_card': 'Intel® HD Graphics 620',
      'hard_disk': '1 TB 7200 rpm SATA',
      'included_items': 'HP Original Bag  (Worth ₹1,499) (#5DD44PA)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '7th Generation Intel® Core™ i3 processor (i3-7020U)',
      'processor_company': 'Intel',
      'processor_type': 'i3',
      'ram': '4 GB DDR4-2133 SDRAM (1 x 4 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ZBook 17 G5 Mobile Workstation',
      'actual_price': '667,152',
      'discout': '224,481',
      'display': None,
      'final_price': '442,671',
      'generation': '7',
      'graphic_card': 'NVIDIA® Quadro® P4200 (8 GB GDDR5 dedicated)',
      'hard_disk': '512 GB PCIe® NVMe™ SSD',
      'included_items': '',
      'os_installed': 'Windows 10 Pro 64',
      'processor': 'Intel® Xeon® processor',
      'processor_company': 'Intel',
      'processor_type': 'Xeon®',
      'ram': '32 GB DDR4-2666 ECC SDRAM (2 X 16 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ZBook 15v G5 Mobile Workstation',
      'actual_price': '183,207',
      'discout': '66,023',
      'display': None,
      'final_price': '117,184',
      'generation': '8',
      'graphic_card': 'NVIDIA® Quadro® P600 (4 GB GDDR5 dedicated)',
      'hard_disk': None,
      'included_items': '',
      'os_installed': 'FreeDOS 2.0',
      'processor': '8th Generation Intel® Core™ i7 processor',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '8 GB DDR4-2666 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ZBook 15u G5 Mobile Workstation',
      'actual_price': '162,354',
      'discout': '37,144',
      'display': None,
      'final_price': '125,210',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '512 GB PCIe® NVMe™ SSD',
      'included_items': '',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i7 processor',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ZBook 15u G5 Mobile Workstation',
      'actual_price': '169,300',
      'discout': '56,606',
      'display': None,
      'final_price': '112,694',
      'generation': '8',
      'graphic_card': 'Intel® HD Graphics 630',
      'hard_disk': '512 GB PCIe® NVMe™ SSD',
      'included_items': '',
      'os_installed': 'FreeDOS 2.0',
      'processor': '8th Generation Intel® Core™ i7 processor',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ZBook 15u G5 Mobile Workstation',
      'actual_price': '166,690',
      'discout': '57,304',
      'display': None,
      'final_price': '109,386',
      'generation': '8',
      'graphic_card': 'AMD Radeon™ Pro WX 3100 Graphics (2 GB GDDR5 dedicated)',
      'hard_disk': '512 GB PCIe® NVMe™ SSD',
      'included_items': '',
      'os_installed': 'FreeDOS 2.0',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP EliteBook x360 1030 G3 Notebook PC',
      'actual_price': '159,347',
      'discout': '1,392',
      'display': None,
      'final_price': '157,955',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '512 GB SSD',
      'included_items': 'HP Active Pen (Worth ₹3,145),HP Original Bag  (Worth ₹1,499) (#5DD44PA)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2133 SDRAM (onboard)',
      'rating': '0.0000'},
     {'Name': 'HP EliteBook x360 1030 G3 Notebook PC',
      'actual_price': '204,403',
      'discout': '1,785',
      'display': None,
      'final_price': '202,618',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '1 TB SSD',
      'included_items': 'HP Active Pen (Worth ₹3,145),HP Original Bag  (Worth ₹1,499) (#5DD44PA)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i7 processor',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '16 GB DDR4-2133 SDRAM (onboard)',
      'rating': '0.0000'},
     {'Name': 'HP ZBook 14u G5 Mobile Workstation',
      'actual_price': '275,830',
      'discout': '114,313',
      'display': None,
      'final_price': '161,517',
      'generation': '8',
      'graphic_card': 'AMD Radeon™ Pro WX 3100 Graphics (2 GB GDDR5 dedicated)',
      'hard_disk': '512 GB SATA SSD',
      'included_items': '',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i7 processor',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '16 GB DDR4-2400 SDRAM (1 x 16 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ZBook 15u G5 Mobile Workstation',
      'actual_price': '206,659',
      'discout': '80,385',
      'display': None,
      'final_price': '126,274',
      'generation': '8',
      'graphic_card': 'AMD Radeon™ Pro WX 3100 Graphics (2 GB GDDR5 dedicated)',
      'hard_disk': '512 GB SSD',
      'included_items': '',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ZBook 14u G5 Mobile Workstation',
      'actual_price': '252,946',
      'discout': '102,177',
      'display': None,
      'final_price': '150,769',
      'generation': '8',
      'graphic_card': 'AMD Radeon™ Pro WX 3100 Graphics (2 GB GDDR5 dedicated)',
      'hard_disk': '512 GB SATA SSD',
      'included_items': '',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i7 processor',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '16 GB DDR4-2400 SDRAM (1 x 16 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ZBook 14u G5 Mobile Workstation',
      'actual_price': '237,270',
      'discout': '94,827',
      'display': None,
      'final_price': '142,443',
      'generation': '8',
      'graphic_card': 'AMD Radeon™ Pro WX 3100 Graphics (2 GB GDDR5 dedicated)',
      'hard_disk': '512 GB SATA SSD',
      'included_items': '',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i7 processor',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ZBook 14u G5 Mobile Workstation',
      'actual_price': '217,955',
      'discout': '91,681',
      'display': None,
      'final_price': '126,274',
      'generation': '8',
      'graphic_card': 'AMD Radeon™ Pro WX 3100 Graphics (2 GB GDDR5 dedicated)',
      'hard_disk': '512 GB SATA SSD',
      'included_items': '',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ZBook 15u G5 Mobile Workstation',
      'actual_price': '249,071',
      'discout': '96,031',
      'display': None,
      'final_price': '153,040',
      'generation': '8',
      'graphic_card': 'AMD Radeon™ Pro WX 3100 Graphics (2 GB GDDR5 dedicated)',
      'hard_disk': '512 GB PCIe® NVMe™ SSD',
      'included_items': '',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i7 processor',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ZBook 14u G5 Mobile Workstation',
      'actual_price': '260,367',
      'discout': '107,327',
      'display': None,
      'final_price': '153,040',
      'generation': '8',
      'graphic_card': 'AMD Radeon™ Pro WX 3100 Graphics (2 GB GDDR5 dedicated)',
      'hard_disk': '512 GB SATA SSD',
      'included_items': '',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i7 processor',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ProBook 440 G5 Notebook PC',
      'actual_price': '64,627',
      'discout': '4,148',
      'display': None,
      'final_price': '60,479',
      'generation': '7',
      'graphic_card': 'Intel® HD Graphics 620',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Business Backpack (Worth ₹4,200)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '7th Generation Intel® Core™ i3 processor',
      'processor_company': 'Intel',
      'processor_type': 'i3',
      'ram': '4 GB DDR4-2400 SDRAM (1 x 4 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ProBook 430 G5 Notebook PC',
      'actual_price': '97,779',
      'discout': '2,894',
      'display': None,
      'final_price': '94,885',
      'generation': '7',
      'graphic_card': 'Intel® HD Graphics 620',
      'hard_disk': '512 GB PCIe® NVMe™ M.2 SSD',
      'included_items': 'HP Business Backpack (Worth ₹4,200)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '7th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP EliteBook 735 G5 Notebook PC',
      'actual_price': '106,227',
      'discout': '928',
      'display': None,
      'final_price': '105,299',
      'generation': None,
      'graphic_card': 'AMD Radeon™ Vega Graphics',
      'hard_disk': '512 GB SSD',
      'included_items': 'HP Business Backpack (Worth ₹4,200)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': 'AMD® Ryzen™ APU processor',
      'processor_company': 'AMD',
      'processor_type': 'Ryzen™ APU',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP EliteBook 1050 G1 Notebook PC',
      'actual_price': '245,747',
      'discout': '2,147',
      'display': None,
      'final_price': '243,600',
      'generation': '8',
      'graphic_card': 'NVIDIA® GeForce® GTX 1050 (4 GB DDR5 dedicated)',
      'hard_disk': '1 TB SSD',
      'included_items': 'HP Business Backpack (Worth ₹4,200)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i7 processor (i7-8750H)',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '16 GB DDR4-2666 SDRAM (1 x 16 GB)',
      'rating': '5.0000'},
     {'Name': 'HP ZBook 15u G5 Mobile Workstation',
      'actual_price': '241,510',
      'discout': '90,741',
      'display': None,
      'final_price': '150,769',
      'generation': '8',
      'graphic_card': 'AMD Radeon™ Pro WX 3100 Graphics (2 GB GDDR5 dedicated)',
      'hard_disk': '512 GB PCIe® NVMe™ SSD',
      'included_items': 'HP Original Bag  (Worth ₹1,499) (#5DD44PA),3 Years Warranty',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i7 processor',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '16 GB DDR4-2400 SDRAM (1 x 16 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ZBook 15u G5 Mobile Workstation',
      'actual_price': '225,835',
      'discout': '83,392',
      'display': None,
      'final_price': '142,443',
      'generation': '8',
      'graphic_card': 'AMD Radeon™ Pro WX 3100 Graphics (2 GB GDDR5 dedicated)',
      'hard_disk': '512 GB PCIe® NVMe™ SSD',
      'included_items': 'HP Business Backpack (Worth ₹4,200)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i7 processor',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ZBook Studio G5 Mobile Workstation',
      'actual_price': '314,632',
      'discout': '93,052',
      'display': None,
      'final_price': '221,580',
      'generation': '8',
      'graphic_card': 'NVIDIA® Quadro® P1000 (4 GB GDDR5 dedicated)',
      'hard_disk': '512 GB PCIe® NVMe™ SSD',
      'included_items': 'HP Business Backpack (Worth ₹4,200)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i7 processor',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '16 GB DDR4-2666 SDRAM (1 x 16 GB)',
      'rating': '3.0000'},
     {'Name': 'HP ZBook 15 G5 Mobile Workstation',
      'actual_price': '350,406',
      'discout': '124,853',
      'display': None,
      'final_price': '225,553',
      'generation': None,
      'graphic_card': 'NVIDIA® Quadro® P2000 (4 GB GDDR5 dedicated)',
      'hard_disk': '1 TB 7200 rpm SATA + 512 GB SSD',
      'included_items': 'HP Business Backpack (Worth ₹4,200)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': None,
      'processor_company': None,
      'processor_type': None,
      'ram': '16 GB (1x16 GB) DDR4 2666',
      'rating': '0.0000'},
     {'Name': 'HP ZBook 15 G5 Mobile Workstation',
      'actual_price': '316,190',
      'discout': '110,771',
      'display': None,
      'final_price': '205,419',
      'generation': None,
      'graphic_card': 'NVIDIA® Quadro® P1000 (4 GB GDDR5 dedicated)',
      'hard_disk': '1 TB 7200 rpm SATA + 128 GB SSD',
      'included_items': 'HP Business Backpack (Worth ₹4,200)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': None,
      'processor_company': None,
      'processor_type': None,
      'ram': '8 GB DDR4-2666 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ZBook 15 G5 Mobile Workstation',
      'actual_price': '240,765',
      'discout': '83,941',
      'display': None,
      'final_price': '156,824',
      'generation': None,
      'graphic_card': 'NVIDIA® Quadro® P1000 (4 GB GDDR5 dedicated)',
      'hard_disk': '1 TB 7200 rpm SATA',
      'included_items': 'HP Business Backpack (Worth ₹4,200)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': None,
      'processor_company': None,
      'processor_type': None,
      'ram': '8 GB DDR4-2666 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ZBook x2 G4 Detachable Workstation',
      'actual_price': '408,283',
      'discout': '145,897',
      'display': None,
      'final_price': '262,386',
      'generation': None,
      'graphic_card': 'NVIDIA® Quadro® M620 (2 GB GDDR5 dedicated)',
      'hard_disk': '512 GB SSD',
      'included_items': 'HP Business Backpack (Worth ₹4,200)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': None,
      'processor_company': None,
      'processor_type': None,
      'ram': '16 GB DDR4-2133 SDRAM (2 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ZBook x2 G4 Detachable Workstation',
      'actual_price': '369,052',
      'discout': '128,360',
      'display': None,
      'final_price': '240,692',
      'generation': None,
      'graphic_card': 'NVIDIA® Quadro® M620 (2 GB GDDR5 dedicated)',
      'hard_disk': '512 GB SSD',
      'included_items': 'HP Business Backpack (Worth ₹4,200)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': None,
      'processor_company': None,
      'processor_type': None,
      'ram': '8 GB DDR4-2133 SDRAM (2 x 4 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ZBook Studio x360 G5 Convertible Workstation',
      'actual_price': '400,665',
      'discout': '144,380',
      'display': None,
      'final_price': '256,285',
      'generation': None,
      'graphic_card': 'NVIDIA® Quadro® P1000 (4 GB GDDR5 dedicated)',
      'hard_disk': '1 TB SSD',
      'included_items': 'HP Business Backpack (Worth ₹4,200)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': None,
      'processor_company': None,
      'processor_type': None,
      'ram': '16 GB DDR4-2666 SDRAM (1 x 16 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ZBook Studio x360 G5 Convertible Workstation',
      'actual_price': '315,831',
      'discout': '100,375',
      'display': None,
      'final_price': '215,456',
      'generation': '8',
      'graphic_card': 'NVIDIA® Quadro® P1000 (4 GB GDDR5 dedicated)',
      'hard_disk': '512 GB PCIe® NVMe™ SSD',
      'included_items': 'HP Business Backpack (Worth ₹4,200)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2666 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ZBook Studio G5 Mobile Workstation',
      'actual_price': '254,780',
      'discout': '89,781',
      'display': None,
      'final_price': '164,999',
      'generation': None,
      'graphic_card': 'NVIDIA® Quadro® P1000 (4 GB GDDR5 dedicated)',
      'hard_disk': '512 GB SSD',
      'included_items': 'HP Business Backpack (Worth ₹4,200)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': None,
      'processor_company': None,
      'processor_type': None,
      'ram': '8 GB DDR4-2666 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP Notebook - 15-da0296tu',
      'actual_price': '35,197',
      'discout': '3,286',
      'display': None,
      'final_price': '31,911',
      'generation': '7',
      'graphic_card': 'Intel® HD Graphics 620',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Original Laptop Bag (Worth ₹1,123)',
      'os_installed': 'FreeDOS 2.0',
      'processor': '7th Generation Intel® Core™ i3 processor',
      'processor_company': 'Intel',
      'processor_type': 'i3',
      'ram': '4 GB DDR4-2133 SDRAM (1 x 4 GB)',
      'rating': '4.0000'},
     {'Name': 'HP Notebook - 15-da0074tx',
      'actual_price': '46,010',
      'discout': '6,516',
      'display': None,
      'final_price': '39,494',
      'generation': '7',
      'graphic_card': 'NVIDIA® GeForce® MX110 (2 GB DDR3 dedicated)',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Original Laptop Bag (Worth ₹1,123)',
      'os_installed': 'FreeDOS 2.0',
      'processor': '7th Generation Intel® Core™ i3 processor',
      'processor_company': 'Intel',
      'processor_type': 'i3',
      'ram': '8 GB DDR4-2133 SDRAM (1 x 8 GB)',
      'rating': '5.0000'},
     {'Name': 'HP Notebook - 15-da0300tu',
      'actual_price': '47,294',
      'discout': '6,743',
      'display': None,
      'final_price': '40,551',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Original Laptop Bag (Worth ₹1,123)',
      'os_installed': 'FreeDOS 2.0',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '4 GB DDR4-2400 SDRAM (1 x 4 GB)',
      'rating': '5.0000'},
     {'Name': 'HP Notebook - 15-da0077tx',
      'actual_price': '57,786',
      'discout': '7,065',
      'display': None,
      'final_price': '50,721',
      'generation': '8',
      'graphic_card': 'NVIDIA® GeForce® MX110 (2 GB GDDR5 dedicated)',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Original Laptop Bag (Worth ₹1,123),1 Year On-site Warranty',
      'os_installed': 'FreeDOS 2.0',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '5.0000'},
     {'Name': 'HP ENVY - 13-ah0044tx',
      'actual_price': '102,232',
      'discout': '5,242',
      'display': None,
      'final_price': '96,990',
      'generation': '8',
      'graphic_card': 'NVIDIA® GeForce® MX150 (2 GB GDDR5 dedicated)',
      'hard_disk': '256 GB SSD',
      'included_items': '',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': '8th Generation Intel® Core™ i7 processor',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '8 GB LPDDR3-2133 SDRAM (onboard)',
      'rating': '4.0000'},
     {'Name': 'HP ENVY - 13-ah0043tx',
      'actual_price': '88,742',
      'discout': '4,352',
      'display': None,
      'final_price': '84,390',
      'generation': '8',
      'graphic_card': 'NVIDIA® GeForce® MX150 (2 GB GDDR5 dedicated)',
      'hard_disk': '256 GB SSD',
      'included_items': '1 Year On-site Warranty',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB LPDDR3-2133 SDRAM (onboard)',
      'rating': '5.0000'},
     {'Name': 'HP ENVY - 13-ah0042tu',
      'actual_price': '64,011',
      'discout': '3,631',
      'display': None,
      'final_price': '60,380',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '128 GB SSD',
      'included_items': '1 Year On-site Warranty',
      'os_installed': 'Windows 10 Home Single Language 64',
      'processor': '8th Generation Intel® Core™ i3 processor',
      'processor_company': 'Intel',
      'processor_type': 'i3',
      'ram': '4 GB LPDDR3-1866 SDRAM (onboard)',
      'rating': '0.0000'},
     {'Name': 'HP EliteBook 830 G5 Notebook PC',
      'actual_price': '107,507',
      'discout': '939',
      'display': None,
      'final_price': '106,568',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '256 GB SSD',
      'included_items': '',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ProBook 640 G4 Notebook PC',
      'actual_price': '95,603',
      'discout': '2,830',
      'display': None,
      'final_price': '92,773',
      'generation': None,
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '1 TB 7200 rpm SATA',
      'included_items': 'HP Overnighter Backpack (Worth ₹2,499)',
      'os_installed': '8th Generation Intel® Core™ i5 processor',
      'processor': None,
      'processor_company': None,
      'processor_type': None,
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ProBook 640 G4 Notebook PC',
      'actual_price': '106,867',
      'discout': '3,164',
      'display': None,
      'final_price': '103,703',
      'generation': None,
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '1 TB 7200 rpm SATA',
      'included_items': 'HP Overnighter Backpack (Worth ₹2,499)',
      'os_installed': 'Intel Core i7-8550U Processor',
      'processor': None,
      'processor_company': None,
      'processor_type': None,
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP 348 G3 Notebook PC',
      'actual_price': '51,827',
      'discout': '2,571',
      'display': None,
      'final_price': '49,256',
      'generation': '6',
      'graphic_card': 'Intel® HD Graphics 520',
      'hard_disk': '1 TB 7200 rpm SATA',
      'included_items': 'HP Original Bag  (Worth ₹1,499) (#5DD44PA)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '6th Generation Intel® Core™ i3 processor',
      'processor_company': 'Intel',
      'processor_type': 'i3',
      'ram': '4 GB DDR4-2133 SDRAM (1 x 4 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ProBook x360 440 G1 Notebook PC',
      'actual_price': '116,211',
      'discout': '3,440',
      'display': '14" FHD Touchscreen',
      'final_price': '112,771',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '512 GB SSD',
      'included_items': 'HP Overnighter Backpack (Worth ₹2,499)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i7 processor',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '1.0000'},
     {'Name': 'HP ZBook 15u G4 Mobile Workstation',
      'actual_price': '176,453',
      'discout': '5,453',
      'display': None,
      'final_price': '171,000',
      'generation': None,
      'graphic_card': 'AMD FirePro™ W4190M Graphics (2 GB GDDR5 dedicated)',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': '',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '16 GB DDR4-2133 SDRAM (1 x 16 GB)',
      'processor_company': None,
      'processor_type': None,
      'ram': None,
      'rating': '0.0000'},
     {'Name': 'HP 245 G5 Notebook PC',
      'actual_price': '20,999',
      'discout': '0',
      'display': None,
      'final_price': '20,999',
      'generation': None,
      'graphic_card': 'AMD Radeon™ R4',
      'hard_disk': '500 GB 5400 rpm SATA',
      'included_items': '',
      'os_installed': 'Free DOS',
      'processor': 'AMD A6 APU',
      'processor_company': 'AMD',
      'processor_type': 'A6 APU',
      'ram': '4 GB DDR3L-1600 SDRAM (1 x 4 GB)',
      'rating': '3.5000'},
     {'Name': 'HP ProBook 440 G5 Notebook PC',
      'actual_price': '83,699',
      'discout': '2,477',
      'display': '35.56 cm(14) diagonal HD LED-backlit touch screen (1366 x 768)',
      'final_price': '81,222',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Overnighter Backpack (Worth ₹2,499)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '4 GB DDR4-2400 SDRAM (1 x 4 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ProBook 430 G5 Notebook PC',
      'actual_price': '87,923',
      'discout': '2,603',
      'display': None,
      'final_price': '85,320',
      'generation': '8',
      'graphic_card': 'Intel® UHD graphics 620',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Overnighter Backpack (Worth ₹2,499)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i7 processor',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '1.0000'},
     {'Name': 'HP 348 G4 Notebook PC',
      'actual_price': '65,907',
      'discout': '3,270',
      'display': None,
      'final_price': '62,637',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '1 TB 7200 rpm SATA',
      'included_items': 'HP Original Bag  (Worth ₹1,499) (#5DD44PA)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ProBook 450 G6 Notebook PC',
      'actual_price': '92,147',
      'discout': '2,728',
      'display': None,
      'final_price': '89,419',
      'generation': '8',
      'graphic_card': 'NVIDIA® GeForce® MX130 (2 GB DDR5 dedicated)',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Overnighter Backpack (Worth ₹2,499)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i7 processor',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ProBook 440 G6 Notebook PC',
      'actual_price': '83,187',
      'discout': '2,462',
      'display': None,
      'final_price': '80,725',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Overnighter Backpack (Worth ₹2,499)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i5 processor (i5-8265U)',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ProBook 430 G6 Notebook PC',
      'actual_price': '86,387',
      'discout': '2,557',
      'display': None,
      'final_price': '83,830',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Overnighter Backpack (Worth ₹2,499)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i5 processor (i5-8265U)',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP Elite x2 1012 G1 Tablet (ENERGY STAR)',
      'actual_price': '136,356',
      'discout': '51,356',
      'display': '12" diagonal FHD UWVA eDP ultra-slim LED-backlit touch screen (1920 x1280), direct bonded, Corning® Gorilla® Glass 4',
      'final_price': '85,000',
      'generation': '6',
      'graphic_card': 'Intel® HD Graphics 515',
      'hard_disk': '128 GB M.2 SATA TLC SSD',
      'included_items': 'Free HP Business Top Load Case (Worth ₹1,775) (#H5M92AA)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '6th Generation Intel® Core™ m5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '4 GB LPDDR3-1866 SDRAM',
      'rating': '0.0000'},
     {'Name': 'HP Chromebook 11A G6 EE',
      'actual_price': '28,320',
      'discout': '0',
      'display': None,
      'final_price': '28,320',
      'generation': None,
      'graphic_card': 'AMD Radeon™ R4 Graphics',
      'hard_disk': '16 GB eMMC',
      'included_items': '',
      'os_installed': 'Chrome OS™ 64',
      'processor': 'AMD A4-Series APU processor',
      'processor_company': 'AMD',
      'processor_type': 'A4-Series APU',
      'ram': '4 GB DDR4-2666 SDRAM (onboard)',
      'rating': '0.0000'},
     {'Name': 'OMEN by HP 15-dc1006tx',
      'actual_price': '158,841',
      'discout': '6,966',
      'display': None,
      'final_price': '151,875',
      'generation': '8',
      'graphic_card': 'NVIDIA® GeForce® RTX 2060 Graphics\xa0(6 GB GDDR6 dedicated)',
      'hard_disk': None,
      'included_items': 'Omen Gaming Bag (Worth ₹4,950)',
      'os_installed': 'Windows 10 Home 64',
      'processor': '8th Generation Intel® Core™ i7 processor',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '16 GB DDR4-2666 SDRAM (1 x 16 GB)',
      'rating': '5.0000'},
     {'Name': 'HP 240 G6 Notebook PC',
      'actual_price': '60,019',
      'discout': '2,978',
      'display': None,
      'final_price': '57,041',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Original Bag  (Worth ₹1,499) (#5DD44PA)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '4 GB DDR4-2400 SDRAM (1 x 4 GB)',
      'rating': '0.0000'},
     {'Name': 'HP 240 G6 Notebook PC',
      'actual_price': '49,907',
      'discout': '2,476',
      'display': None,
      'final_price': '47,431',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Original Bag  (Worth ₹1,499) (#5DD44PA)',
      'os_installed': 'FreeDOS 2.0',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '4 GB DDR4-2400 SDRAM (1 x 4 GB)',
      'rating': '0.0000'},
     {'Name': 'HP Spectre Folio - 13-ak0040tu',
      'actual_price': '214,110',
      'discout': '13,235',
      'display': '13.3" diagonal FHD IPS BrightView WLED-backlit micro-edge multitouch-enabled edge-to-edge glass (1920 x 1080)',
      'final_price': '200,875',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 615',
      'hard_disk': '512 GB PCIe® NVMe™ M.2 SSD',
      'included_items': '',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i7 processor',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '16 GB LPDDR3-1866 SDRAM (onboard)',
      'rating': '5.0000'},
     {'Name': 'HP 250 G6 Notebook PC',
      'actual_price': '37,200',
      'discout': '0',
      'display': None,
      'final_price': '37,200',
      'generation': '7',
      'graphic_card': 'Intel® HD Graphics 620',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': '',
      'os_installed': 'Windows 10 Home Single Language 64 – HP recommends Windows 10 Pro.',
      'processor': '7th Generation Intel® Core™ i3 processor',
      'processor_company': 'Intel',
      'processor_type': 'i3',
      'ram': '4 GB DDR4-2133 SDRAM (1 x 4 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ZBook Studio x360 G5 Convertible Workstation',
      'actual_price': '437,143',
      'discout': '95,703',
      'display': None,
      'final_price': '341,440',
      'generation': '7',
      'graphic_card': 'NVIDIA® Quadro® P1000 (4 GB GDDR5 dedicated)',
      'hard_disk': '512 GB PCIe® NVMe™ SSD',
      'included_items': '',
      'os_installed': 'Windows 10 Pro 64',
      'processor': 'Intel® Xeon® processor',
      'processor_company': 'Intel',
      'processor_type': 'Xeon®',
      'ram': '16 GB DDR4-2666 SDRAM (1 x 16 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ZBook 17 G5 Mobile Workstation',
      'actual_price': '353,597',
      'discout': '116,076',
      'display': None,
      'final_price': '237,521',
      'generation': '8',
      'graphic_card': 'NVIDIA® Quadro® P1000 (4 GB GDDR5 dedicated)',
      'hard_disk': '512 GB PCIe® NVMe™ SSD',
      'included_items': '',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i7 processor',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '16 GB DDR4-2666 SDRAM (1 x 16 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ZBook 15v G5 Mobile Workstation',
      'actual_price': '337,097',
      'discout': '97,546',
      'display': None,
      'final_price': '239,551',
      'generation': '8',
      'graphic_card': 'NVIDIA® Quadro® P600 (4 GB GDDR5 dedicated)',
      'hard_disk': '512GB PCIe NVMe TLC SSD',
      'included_items': '',
      'os_installed': 'Windows 10 Pro 64',
      'processor': 'Intel® Xeon® E3 processor',
      'processor_company': 'Intel',
      'processor_type': 'Xeon® E3',
      'ram': '16 GB DDR4-2666 SDRAM (1 x 16 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ZBook 15 G5 Mobile Workstation',
      'actual_price': '347,045',
      'discout': '76,076',
      'display': None,
      'final_price': '270,969',
      'generation': '8',
      'graphic_card': 'NVIDIA® Quadro® P2000 (4 GB GDDR5 dedicated)',
      'hard_disk': '512 GB PCIe® NVMe™ SSD',
      'included_items': '',
      'os_installed': 'Windows 10 Pro 64',
      'processor': 'Intel® Xeon® processor',
      'processor_company': 'Intel',
      'processor_type': 'Xeon®',
      'ram': '16 GB DDR4-2666 SDRAM (1 x 16 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ZBook 14u G5 Mobile Workstation',
      'actual_price': '164,746',
      'discout': '52,052',
      'display': None,
      'final_price': '112,694',
      'generation': '8',
      'graphic_card': 'Intel® HD Graphics 620',
      'hard_disk': '512 GB HP Z Turbo Drive PCIe® SSD',
      'included_items': '',
      'os_installed': 'FreeDOS 2.0',
      'processor': '8th Generation Intel® Core™ i7 processor',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP 245 G6',
      'actual_price': '24,200',
      'discout': '0',
      'display': None,
      'final_price': '24,200',
      'generation': None,
      'graphic_card': None,
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': '',
      'os_installed': 'FreeDOS',
      'processor': 'BU IDS UMA A9-9425',
      'processor_company': None,
      'processor_type': None,
      'ram': '4GB (1x4GB) 1866 DDR4',
      'rating': '0.0000'},
     {'Name': 'HP EliteBook x360 1030 G3 Notebook PC',
      'actual_price': '143,347',
      'discout': '1,253',
      'display': None,
      'final_price': '142,094',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '256 GB SSD',
      'included_items': 'HP Active Pen (Worth ₹3,145),HP Original Bag  (Worth ₹1,499) (#5DD44PA)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB DDR4-2133 SDRAM (onboard)',
      'rating': '0.0000'},
     {'Name': 'HP 250 G6 Notebook PC',
      'actual_price': '40,947',
      'discout': '2,032',
      'display': None,
      'final_price': '38,915',
      'generation': '7',
      'graphic_card': 'AMD Radeon™ 520 Graphics (2 GB GDDR5 dedicated)',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Original Bag  (Worth ₹1,499) (#5DD44PA)',
      'os_installed': 'FreeDOS 2.0',
      'processor': '7th Generation Intel® Core™ i3 processor',
      'processor_company': 'Intel',
      'processor_type': 'i3',
      'ram': '4 GB DDR4-2133 SDRAM (1 x 4 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ProBook 645 G4 Notebook PC',
      'actual_price': '79,603',
      'discout': '2,357',
      'display': None,
      'final_price': '77,246',
      'generation': None,
      'graphic_card': 'AMD Radeon™ Vega Graphics',
      'hard_disk': '1 TB 7200 rpm SATA',
      'included_items': 'HP Overnighter Backpack (Worth ₹2,499)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': 'AMD® Ryzen™ APU processor',
      'processor_company': 'AMD',
      'processor_type': 'Ryzen™ APU',
      'ram': '8 GB DDR4-2400 SDRAM (1 x 8 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ZBook 15u G5 Mobile Workstation',
      'actual_price': '264,534',
      'discout': '103,017',
      'display': None,
      'final_price': '161,517',
      'generation': None,
      'graphic_card': 'AMD Radeon™ Pro WX 3100 Graphics (2 GB GDDR5 dedicated)',
      'hard_disk': '512 GB SSD',
      'included_items': 'HP Business Backpack (Worth ₹4,200)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': None,
      'processor_company': None,
      'processor_type': None,
      'ram': '16 GB DDR4-2400 SDRAM (1 x 16 GB)',
      'rating': '0.0000'},
     {'Name': 'HP EliteBook x360 1030 G3 Notebook PC',
      'actual_price': '208,728',
      'discout': '2,310',
      'display': 'HP Sure View, 13.3" FHD touch screen',
      'final_price': '206,418',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '1 TB SSD',
      'included_items': 'HP Overnighter Backpack (Worth ₹2,499)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i7 processor',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '16 GB LPDDR3-2133 SDRAM (onboard)',
      'rating': '0.0000'},
     {'Name': 'HP Elite x2 1013 G3 Tablet',
      'actual_price': '167,076',
      'discout': '3,602',
      'display': '13" Touchscreen with Corning® Gorilla® Glass 4',
      'final_price': '163,474',
      'generation': '8',
      'graphic_card': 'Intel® UHD Graphics 620',
      'hard_disk': '512 GB SSD',
      'included_items': 'HP Business Backpack (Worth ₹4,200)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '8th Generation Intel® Core™ i5 processor',
      'processor_company': 'Intel',
      'processor_type': 'i5',
      'ram': '8 GB LPDDR3-2133 SDRAM (onboard)',
      'rating': '0.0000'},
     {'Name': 'HP ProBook 445 G2 Notebook PC (ENERGY STAR)',
      'actual_price': '43,990',
      'discout': '490',
      'display': None,
      'final_price': '43,500',
      'generation': None,
      'graphic_card': 'AMD Radeon™ R5',
      'hard_disk': '500 GB 5400 rpm SATA',
      'included_items': 'HP Overnighter Backpack (Worth ₹2,499)',
      'os_installed': 'Windows 7 Professional 64',
      'processor': 'AMD A8 APU',
      'processor_company': 'AMD',
      'processor_type': 'A8 APU',
      'ram': '4 GB DDR3L-1600 SDRAM (1 x 4 GB)',
      'rating': '0.0000'},
     {'Name': 'HP Chromebook 14 G5',
      'actual_price': '36,900',
      'discout': '0',
      'display': '35.56 cm(14) diagonal HD SVA eDP anti-glare LED-backlit (1366 x 768)',
      'final_price': '36,900',
      'generation': None,
      'graphic_card': 'Intel® HD Graphics 500',
      'hard_disk': '64 GB eMMC',
      'included_items': 'HP Original Bag  (Worth ₹1,499) (#5DD44PA)',
      'os_installed': 'Chrome OS™ 64',
      'processor': 'Intel® Celeron® processor',
      'processor_company': 'Intel',
      'processor_type': 'Celeron®',
      'ram': '8 GB LPDDR4-2400 SDRAM (onboard)',
      'rating': '5.0000'},
     {'Name': 'HP Chromebook 14 G5',
      'actual_price': '35,654',
      'discout': '0',
      'display': '35.56 cm(14) diagonal HD SVA eDP anti-glare LED-backlit (1366 x 768)',
      'final_price': '35,654',
      'generation': None,
      'graphic_card': 'Intel® HD Graphics 500',
      'hard_disk': '32 GB eMMC',
      'included_items': 'HP Original Bag  (Worth ₹1,499) (#5DD44PA)',
      'os_installed': 'Chrome OS™ 64',
      'processor': 'Intel® Celeron® processor',
      'processor_company': 'Intel',
      'processor_type': 'Celeron®',
      'ram': '8 GB LPDDR4-2400 SDRAM (onboard)',
      'rating': '0.0000'},
     {'Name': 'HP Chromebook 14 G5',
      'actual_price': '30,749',
      'discout': '0',
      'display': '35.56 cm(14) diagonal HD SVA eDP anti-glare LED-backlit (1366 x 768)',
      'final_price': '30,749',
      'generation': None,
      'graphic_card': 'Intel® HD Graphics 500',
      'hard_disk': '32 GB eMMC',
      'included_items': 'HP Original Bag  (Worth ₹1,499) (#5DD44PA)',
      'os_installed': 'Chrome OS™ 64',
      'processor': 'Intel® Celeron® processor',
      'processor_company': 'Intel',
      'processor_type': 'Celeron®',
      'ram': '4 GB LPDDR4-2400 SDRAM (onboard)',
      'rating': '3.0000'},
     {'Name': 'HP Chromebook 14 G5',
      'actual_price': '30,000',
      'discout': '0',
      'display': '35.56 cm(14) diagonal HD SVA eDP anti-glare LED-backlit (1366 x 768)',
      'final_price': '30,000',
      'generation': None,
      'graphic_card': 'Intel® HD Graphics 500',
      'hard_disk': '16 GB eMMC',
      'included_items': 'HP Original Bag  (Worth ₹1,499) (#5DD44PA)',
      'os_installed': 'Chrome OS™ 64',
      'processor': 'Intel® Celeron® processor',
      'processor_company': 'Intel',
      'processor_type': 'Celeron®',
      'ram': '4 GB LPDDR4-2400 SDRAM (onboard)',
      'rating': '0.0000'},
     {'Name': 'HP EliteBook x360 1020 G2',
      'actual_price': '225,000',
      'discout': '0',
      'display': None,
      'final_price': '225,000',
      'generation': '7',
      'graphic_card': 'Intel® HD Graphics 620',
      'hard_disk': '512 GB PCIe® NVMe™ M.2 SSD',
      'included_items': 'HP Business Backpack (Worth ₹4,200)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '7th Generation Intel® Core™ i7 processor',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '16 GB LPDDR3-1866 SDRAM',
      'rating': '0.0000'},
     {'Name': 'HP EliteBook 1040 G4 Notebook PC',
      'actual_price': '252,000',
      'discout': '0',
      'display': '35.56 cm(14) diagonal FHD IPS eDP LED-backlit touch screen with Corning® Gorilla® Glass 4 and privacy screen, 700 cd/m², 100% sRGB (1920 x 1080)',
      'final_price': '252,000',
      'generation': '7',
      'graphic_card': 'Intel® HD Graphics 630',
      'hard_disk': '1 TB PCIe® NVMe™ SSD',
      'included_items': 'HP Business Backpack (Worth ₹4,200)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': '7th Generation Intel® Core™ i7 processor',
      'processor_company': 'Intel',
      'processor_type': 'i7',
      'ram': '16 GB DDR4-2133 SDRAM (onboard)',
      'rating': '0.0000'},
     {'Name': 'HP 250 G6 Notebook PC',
      'actual_price': '34,810',
      'discout': '200',
      'display': None,
      'final_price': '34,610',
      'generation': '6',
      'graphic_card': 'Intel® HD Graphics 520',
      'hard_disk': '1 TB 5400 rpm SATA',
      'included_items': 'HP Original Bag  (Worth ₹1,499) (#5DD44PA)',
      'os_installed': 'FreeDos 2.0',
      'processor': '6th Generation Intel® Core™ i3 processor',
      'processor_company': 'Intel',
      'processor_type': 'i3',
      'ram': '4 GB DDR4-2133 SDRAM (1 x 4 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ProBook 445 G2 Notebook PC (ENERGY STAR)',
      'actual_price': '55,008',
      'discout': '708',
      'display': '14" diagonal HD anti-glare LED-backlit (1366 x 768)',
      'final_price': '54,300',
      'generation': None,
      'graphic_card': 'AMD Radeon™ R7 M260DX (1 GB DDR3 dedicated, dual graphics)',
      'hard_disk': '500 GB 7200 rpm SATA',
      'included_items': 'HP Overnighter Backpack (Worth ₹2,499)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': 'AMD A10 APU',
      'processor_company': 'AMD',
      'processor_type': 'A10 APU',
      'ram': '4 GB DDR3L-1600 SDRAM (1 x 4 GB)',
      'rating': '0.0000'},
     {'Name': 'HP ProBook 445 G2 Notebook PC (ENERGY STAR)',
      'actual_price': '79,000',
      'discout': '1,000',
      'display': '14" diagonal HD anti-glare LED-backlit (1366 x 768)',
      'final_price': '78,000',
      'generation': None,
      'graphic_card': 'AMD Radeon™ R6',
      'hard_disk': '500 GB 5400 rpm SATA',
      'included_items': 'HP Overnighter Backpack (Worth ₹2,499)',
      'os_installed': 'Windows 10 Pro 64',
      'processor': 'AMD A10 APU',
      'processor_company': 'AMD',
      'processor_type': 'A10 APU',
      'ram': '8 GB DDR3L-1600 SDRAM (1 x 8 GB)',
      'rating': '0.0000'}]



creating a dataframe with the List of products


```python
df=pd.DataFrame(prod_list)
df.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Name</th>
      <th>actual_price</th>
      <th>discout</th>
      <th>display</th>
      <th>final_price</th>
      <th>generation</th>
      <th>graphic_card</th>
      <th>hard_disk</th>
      <th>included_items</th>
      <th>os_installed</th>
      <th>processor</th>
      <th>processor_company</th>
      <th>processor_type</th>
      <th>ram</th>
      <th>rating</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>HP ENVY x360 - 13-ag0035au</td>
      <td>83,496</td>
      <td>10,506</td>
      <td>13.3" FHD multitouch-enabled edge-to-edge glas...</td>
      <td>72,990</td>
      <td>None</td>
      <td>AMD Radeon™ Vega 8 Graphics</td>
      <td>256 GB SSD</td>
      <td></td>
      <td>Windows 10 Home Single Language 64</td>
      <td>AMD Ryzen™ 5 processor</td>
      <td>AMD</td>
      <td>Ryzen™ 5</td>
      <td>8 GB DDR4-2400 SDRAM (onboard)</td>
      <td>3.6667</td>
    </tr>
    <tr>
      <th>1</th>
      <td>HP Gaming Pavilion - 15-cx0140tx</td>
      <td>86,476</td>
      <td>13,486</td>
      <td>None</td>
      <td>72,990</td>
      <td>8</td>
      <td>NVIDIA® GeForce® GTX 1050 (4 GB GDDR5 dedicated)</td>
      <td>1 TB 7200 rpm SATA</td>
      <td>HP Odyssey backpack (Worth ₹3,499),Microsoft O...</td>
      <td>Windows 10 Home Single Language 64</td>
      <td>8th Generation Intel® Core™ i5 processor</td>
      <td>Intel</td>
      <td>i5</td>
      <td>8 GB DDR4-2666 SDRAM (1 x 8 GB)</td>
      <td>5.0000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>HP Notebook - 15-da0435tx</td>
      <td>50,292</td>
      <td>5,712</td>
      <td>None</td>
      <td>44,580</td>
      <td>7</td>
      <td>NVIDIA® GeForce® MX110 (2 GB DDR3 dedicated)</td>
      <td>1 TB 5400 rpm SATA</td>
      <td></td>
      <td>Windows 10 Home Single Language 64</td>
      <td>7th Generation Intel® Core™ i3 processor</td>
      <td>Intel</td>
      <td>i3</td>
      <td>8 GB DDR4-2133 SDRAM (1 x 8 GB)</td>
      <td>4.0000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>HP Notebook - 15g-dr0006tx</td>
      <td>66,137</td>
      <td>7,146</td>
      <td>None</td>
      <td>58,991</td>
      <td>8</td>
      <td>NVIDIA® GeForce® MX110 (2 GB DDR3 dedicated)</td>
      <td>1 TB 5400 rpm SATA</td>
      <td>HP Original Laptop Bag (Worth ₹1,123),1 Year O...</td>
      <td>Windows 10 Home Single Language 64</td>
      <td>8th Generation Intel® Core™ i5 processor</td>
      <td>Intel</td>
      <td>i5</td>
      <td>8 GB DDR4-2400 SDRAM (1 x 8 GB)</td>
      <td>4.0625</td>
    </tr>
    <tr>
      <th>4</th>
      <td>HP Notebook 15-da1030tu</td>
      <td>50,720</td>
      <td>3,730</td>
      <td>None</td>
      <td>46,990</td>
      <td>8</td>
      <td>None</td>
      <td>1 TB 5400 rpm SATA</td>
      <td>HP Original Laptop Bag (Worth ₹1,123),Microsof...</td>
      <td>Windows 10 Home 64</td>
      <td>8th Generation Intel® Core™ i5 processor</td>
      <td>Intel</td>
      <td>i5</td>
      <td>4 GB DDR4-2400 SDRAM</td>
      <td>2.0000</td>
    </tr>
  </tbody>
</table>
</div>



saving the dataframe into Hp_laptops.csv


```python
df.to_csv("Hp_laptops.csv")
```


