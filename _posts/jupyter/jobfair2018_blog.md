
### So, for the last few screen shots, you must have heard how bad the website is --- so now scrape away! For the Bot!


```python
import pandas as pd
import re
```

First, I have downloaded the "root" html page, to get all the ids of organizations. 


```python
from bs4 import BeautifulSoup
soup = BeautifulSoup('./www.partners4employment.ca/student-alumni.htm', 'html.parser')
```

    /home/penpen/anaconda3/lib/python3.5/site-packages/bs4/__init__.py:219: UserWarning: "b'./www.partners4employment.ca/student-alumni.htm'" looks like a filename, not markup. You should probably open this file and pass the filehandle into Beautiful Soup.
      ' Beautiful Soup.' % markup)


the reason I didn't use soup for this step is because... it's aon over kill. 


```python
reg_str = 'registrationId : '
len_reg_str = len(reg_str)
ids = []
with open('./www.partners4employment.ca/student-alumni/current-participating-organizations.htm') as f:
    content = f.readlines()
    for l in content: 
        m = re.search('registrationId : ([0-9]+)', l)
        if (m is not None):
            ids = ids + [m.group(0)[len_reg_str:]]

print("number of companies at jobfair: ", len(ids))
```

    number of companies at jobfair:  194


## Sending Requests to get stuff we need


```python
import requests
```

we're using some test data to see how the page should be parsed. 


```python
endpoint = "https://www.partners4employment.ca/student-alumni/current-participating-organizations.htm"
data = {'action':'displayRegInfo', 
       'registrationId': '3279'} 
r = requests.post(url = endpoint, data = data)
job_soup = BeautifulSoup(r.text, "lxml")
```


```python
name = job_soup.find("h1").text.strip()
print("name of company: ", name)
profile = job_soup.find_all("div", class_="controls-text")
counter = 0
for p in profile: 
    print("#", counter)
    print(p.text.strip())
    print("-----------------------------------")
    counter = counter + 1
```

    name of company:  Think Research
    # 0
    78
    -----------------------------------
    # 1
    Think Research is changing the way healthcare's delivered - no, really! - and not in the Buckley's "tastes awful but it works" kind of way. We are building software to give clinicians the information they need to treat patients better and faster. Why Us? It's not every day you get to change the way your friends and family are cared for. Our culture is one of the things we're most proud of--our fun, freindly and talented team will become your second family! Did we mention we're located in the core of Downtown Toronto?
    -----------------------------------
    # 2
    Learn more at www.thinkresearch.com/ca/company/careers/engineering/. 
    
    
    Engineering Co-op Students for the Summer Waterloo Co-op Term ( May 14th start): 4 month co-ops for the summer
    Graduates/Alumni: Ruby Developers, Software Developers (full time)
    -----------------------------------
    # 3
    Full-time|Co-op/Internship|Contract
    -----------------------------------
    # 4
    Ontario (GTA)|Ontario (excluding GTA)
    -----------------------------------
    # 5
    www.linkedin.com/company/1590921/
    -----------------------------------
    # 6
    @TRChealth
    -----------------------------------
    # 7
    www.facebook.com/TRChealth/
    -----------------------------------
    # 8
    156 Front Street West, 5th Floor, Toronto, Ontario M5J 2L6
    416.977.1955
    www.thinkresearch.com/ca
    -----------------------------------


Okay, so we have description, address, name, phone number, etc. 
TL;DR: names and info have to be stripped seperately, but they're there. 


## Actual stripping script: 


```python
company_df = pd.DataFrame(columns=[
    'id', 
    'name', 
    'booth', 
    'profile', 
    'positions', 
    'employment types', 
    'location', 
    'linkedin',
    'twitter',
    'facebook', 
    'contact'
])

for currid in ids: 
    endpoint = "https://www.partners4employment.ca/student-alumni/current-participating-organizations.htm"
    data = {'action':'displayRegInfo', 
           'registrationId': currid}
    r = requests.post(url = endpoint, data = data)
    job_soup = BeautifulSoup(r.text,  "lxml")
    
    name = job_soup.find("h1").text.strip()
    details = job_soup.find_all("div", class_="controls-text")
    boothnum = details[0].text.strip()
    profile = details[1].text.strip()
    positions = details[2].text.strip().split(' - ')
    employmentTypes = details[3].text.strip().split('/')
    location = details[4].text.strip().split('|')
    linkedin = details[5].text.strip()
    twitter = details[6].text.strip()
    facebook = details[7].text.strip()
    contact = details[8].text.strip()
    
    company_df = company_df.append({
        'id': currid,
        'name': name, 
        'booth': boothnum, 
        'profile':profile, 
        'positions':positions, 
        'employment types':employmentTypes, 
        'location': location, 
        'linkedin': linkedin,
        'twitter': twitter,
        'facebook':facebook, 
        'contact':contact
    }, ignore_index=True)
    print("added ", name, " currlen: ", company_df.size)
```

    added  Accedo  currlen:  11
    added  Adastra Corporation  currlen:  22
    added  Aecon  currlen:  33
    added  Aerotek  currlen:  44
    added  African Lion Safari  currlen:  55
    added  Agriculture & Agri-Food Canada  currlen:  66
    added  Andiamo  currlen:  77
    added  Arcane  currlen:  88
    added  Arctic Glacier Canada  currlen:  99
    added  Arvato  currlen:  110
    added  Auvik Networks  currlen:  121
    added  Aviva Canada - Healthcare Claims  currlen:  132
    added  B&R Industrial Automation  currlen:  143
    added  BASF Corporation  currlen:  154
    added  BBM Canada  currlen:  165
    added  Big Viking Games  currlen:  176
    added  BlackBerry Limited  currlen:  187
    added  Bonanza Gardens  currlen:  198
    added  Brock Solutions  currlen:  209
    added  BSM Technologies  currlen:  220
    added  BWXT Canada Ltd.  currlen:  231
    added  Camp Couchiching  currlen:  242
    added  Camp Kennebec  currlen:  253
    added  Camp Kodiak  currlen:  264
    added  Camp Trillium  currlen:  275
    added  Canada Revenue Agency  currlen:  286
    added  Canadian Broadcasting Corporation  currlen:  297
    added  Canadian Coast Guard  currlen:  308
    added  Canadian Coast Guard  currlen:  319
    added  Canadian Deafblind Association Ontario Chapter  currlen:  330
    added  Ceridian  currlen:  341
    added  CF Crozier & Associates  currlen:  352
    added  CGI  currlen:  363
    added  Children's Mental Health Services  currlen:  374
    added  Christian Horizons-West District  currlen:  385
    added  CIHI  currlen:  396
    added  Cintas Canada Ltd  currlen:  407
    added  Clearpath Robotics  currlen:  418
    added  Clio  currlen:  429
    added  CNIB Lake Joseph Centre  currlen:  440
    added  Cole Engineering Group Ltd  currlen:  451
    added  Collabera  currlen:  462
    added  Communications Security Establishment  currlen:  473
    added  Computer Talk Technology  currlen:  484
    added  Crawford and Company (Canada)  currlen:  495
    added  Crestwood Valley Day Camp  currlen:  506
    added  Cummins Canada  currlen:  517
    added  Dalton Associates  currlen:  528
    added  DarkMatter Canada Inc.  currlen:  539
    added  Dealer-FX  currlen:  550
    added  Dejero  currlen:  561
    added  Del Industrial Metals  currlen:  572
    added  Dell  currlen:  583
    added  DESCH Canada Ltd.  currlen:  594
    added  DHL Supply Chain  currlen:  605
    added  Double Negative Canada Productions Ltd.  currlen:  616
    added  DSEL  currlen:  627
    added  Dundas Data Visualization, Inc.  currlen:  638
    added  Dynatrace  currlen:  649
    added  Edison Engineers Inc.  currlen:  660
    added  Edsence International Children's College  currlen:  671
    added  Edward Jones  currlen:  682
    added  EMCO Corporation  currlen:  693
    added  Englobe  currlen:  704
    added  Enterprise Holdings  currlen:  715
    added  Equifax  currlen:  726
    added  ESCRYPT  currlen:  737
    added  eSentire  currlen:  748
    added  ETBO Tool & Die  currlen:  759
    added  FieldCore  currlen:  770
    added  Finastra  currlen:  781
    added  Fluent Home Ltd.  currlen:  792
    added  Fortigo Freight  currlen:  803
    added  Fortinet Technologies  currlen:  814
    added  Fowler Construction  currlen:  825
    added  Fusion Retail Analytics  currlen:  836
    added  General Dynamics Mission Systems-Canada  currlen:  847
    added  General Motors of Canada  currlen:  858
    added  Geotab Inc  currlen:  869
    added  GHD Limited  currlen:  880
    added  goeasy Ltd.  currlen:  891
    added  GoodLife Fitness  currlen:  902
    added  Gordon Food Service  currlen:  913
    added  Guelph Police Service  currlen:  924
    added  Health Canada and the Public Agency of Canada  currlen:  935
    added  HESS International Educational Group  currlen:  946
    added  HollisWealth  currlen:  957
    added  Huawei  currlen:  968
    added  Indellient  currlen:  979
    added  INDIVA Inc.  currlen:  990
    added  InnoSoft Canada Inc.  currlen:  1001
    added  Innovative Automation  currlen:  1012
    added  Insight Global  currlen:  1023
    added  Investors Group  currlen:  1034
    added  Keyence Canada Inc.  currlen:  1045
    added  Kinaxis  currlen:  1056
    added  Klenzoid Canada Inc.  currlen:  1067
    added  KMW Outreach Inc  currlen:  1078
    added  Knowledge First Financial  currlen:  1089
    added  Konica Minolta  currlen:  1100
    added  Konrad Group  currlen:  1111
    added  Labstat International ULC  currlen:  1122
    added  Lafarge Canada Inc  currlen:  1133
    added  Lakeside Produce  currlen:  1144
    added  Libro Credit Union  currlen:  1155
    added  Lixar IT  currlen:  1166
    added  Manulife  currlen:  1177
    added  Manulife Securities Incorporated  currlen:  1188
    added  MCAP  currlen:  1199
    added  McKellar Structured Settlements Inc.  currlen:  1210
    added  McRae Integration  currlen:  1221
    added  Meltwater  currlen:  1232
    added  Mobeewave  currlen:  1243
    added  Mozzaz  currlen:  1254
    added  Mueller Water Products  currlen:  1265
    added  Multi-Health Systems Inc.  currlen:  1276
    added  MultiView Canada  currlen:  1287
    added  Natural Resources Canada  currlen:  1298
    added  Noble Corporation  currlen:  1309
    added  Ontario Drive & Gear Limited  currlen:  1320
    added  Ontario One Call  currlen:  1331
    added  Operis  currlen:  1342
    added  Oracle  currlen:  1353
    added  PCC Aerostructures  currlen:  1364
    added  PCL Constructors Canada Inc.  currlen:  1375
    added  PEER Group  currlen:  1386
    added  Penske Truck Leasing  currlen:  1397
    added  Primerica  currlen:  1408
    added  Princeton Holdings Limited  currlen:  1419
    added  Public Service Commission of Canada  currlen:  1430
    added  Qualicom Innovations Inc.  currlen:  1441
    added  Radium Golf Group  currlen:  1452
    added  Rapid Novor Inc  currlen:  1463
    added  Region of Waterloo  currlen:  1474
    added  Reynolds and Reynolds  currlen:  1485
    added  RidgeTech Automation Inc.  currlen:  1496
    added  RioCan  currlen:  1507
    added  Robert Half Canada  currlen:  1518
    added  Rome Transportation  currlen:  1529
    added  Rothmans, Benson & Hedges - INKOMPASS  currlen:  1540
    added  Royal Adhesives & Sealants Canada Ltd  currlen:  1551
    added  Royal Canadian Mounted Police  currlen:  1562
    added  S&C Electric Company  currlen:  1573
    added  SAP Canada  currlen:  1584
    added  Schaefer Systems International  currlen:  1595
    added  Schaeffler Canada Inc.  currlen:  1606
    added  Schneider Electric  currlen:  1617
    added  Scotlynn Commodities  currlen:  1628
    added  Scribd  currlen:  1639
    added  Septodont  currlen:  1650
    added  SNC-Lavalin  currlen:  1661
    added  Sofina Foods Inc.  currlen:  1672
    added  SOTI  currlen:  1683
    added  Stackpole International  currlen:  1694
    added  StackTeck Systems Ltd  currlen:  1705
    added  Staples Canada  currlen:  1716
    added  Sun Life Financial  currlen:  1727
    added  Synopsys  currlen:  1738
    added  TalentEgg  currlen:  1749
    added  Tangam Gaming Inc.  currlen:  1760
    added  TAO Solutions Inc.  currlen:  1771
    added  TD Bank  currlen:  1782
    added  Teledyne DALSA  currlen:  1793
    added  The Sherwin-Williams Company  currlen:  1804
    added  Think Research  currlen:  1815
    added  Thompsons Limited  currlen:  1826
    added  Thresholds Homes and Supports Inc.  currlen:  1837
    added  Tigercat  currlen:  1848
    added  Timberland Equipment Limited  currlen:  1859
    added  Tjene  currlen:  1870
    added  Toronto Police Service  currlen:  1881
    added  Toyota Motor Manufacturing Canada Inc.  currlen:  1892
    added  TRADER Corporation  currlen:  1903
    added  TransUnion  currlen:  1914
    added  Trend Hunter Inc.  currlen:  1925
    added  Uline Shipping Supplies  currlen:  1936
    added  Ultimate Software  currlen:  1947
    added  Value Connect Inc.  currlen:  1958
    added  Vena Solutions  currlen:  1969
    added  Verafin  currlen:  1980
    added  Viking-Cives  currlen:  1991
    added  WalterFedy  currlen:  2002
    added  Ward & Uptigrove Chartered Professional Accountants  currlen:  2013
    added  Waterloo Regional Police Service  currlen:  2024
    added  WebSan Solutions Inc.  currlen:  2035
    added  Weishaupt Corporation  currlen:  2046
    added  Wells Fargo  currlen:  2057
    added  WIS International  currlen:  2068
    added  World Wide Logistics Inc.  currlen:  2079
    added  WorleyParsons Canada  currlen:  2090
    added  X by 2  currlen:  2101
    added  YMCA Summer Work Student Exchange  currlen:  2112
    added  York Regional Police  currlen:  2123
    added  ZTR Control Systems  currlen:  2134

