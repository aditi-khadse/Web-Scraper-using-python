# Web-Scraper-using-python
How to design a web scraper to read articles off theverge.com using Python
The script will be able to perform the following:
- Reading the headline, get the link of the article, the author, and the date of each of the articles
found on "theverge.com"
- Storing these in a CSV file titled `ddmmyyy_verge.csv`, with the following header `id, URL,
headline, author, date`.
- Creating an SQLite database to store the same data, and make sure that the id is the primary
key

## Import the required modules in python.
Requests and bs4 are not built in modules.
Therefore we need to install them first.
```
import requests
from bs4 import BeautifulSoup
import pandas as pd
import sqlite3
from datetime import datetime
import time
```
## For scraping the information
```
# define the url of the website
url = 'https://www.theverge.com/'

# send a GET request to the website
response = requests.get(url)

# use BeautifulSoup to parse the HTML content
soup = BeautifulSoup(response.content, 'html.parser')

# find all the articles in the page
articles_1 = soup.find_all("li",{"class":"relative"})

#create an empty list to store the data
data=[]

# iterate over the articles to extract information
for index, li in enumerate(articles_1):
    # extract the headline
    headline = li.h2.a.text.strip()
    print(headline)
    
    # extract the link of the article
    link = li.h2.a['href']
    print(link)
    
    # extract the author of the article
    author = li.find('a', class_='text-gray-31 hover:shadow-underline-inherit dark:text-franklin mr-8').text.strip()
    print(author)
    
    # extract the date of the article
    date = li.find('span', class_='text-gray-63 dark:text-gray-94').text.strip()
    print(date)

    print()
```

## Forming a CSV file
```
# in the same loop
# create a dictionary to store the data
    article_data = {
        'id': f'{index + 1}',
        'URL': link,
        'headline': headline,
        'author': author,
        'date': date
    }

    # append the dictionary to the list
    data.append(article_data)

    #create a dataframe from the list of dictionaries
    df = pd.DataFrame(data)

    #define the filename
    filename = f'{pd.Timestamp.now().strftime("%d%m%Y")}_verge.csv'

    #write the dataframe to a csv file
    df.to_csv(filename, index=False)
```
## Creating a SQlite database
```
#continuing the loop
#sqlite database
    db_filename = datetime.today().strftime("%d%m%Y") + "_verge.db"

    # Define the headers for the CSV file
    csv_headers = ['id', 'URL', 'headline', 'author', 'date']

    # Create a connection to the SQLite database
    conn = sqlite3.connect(db_filename)
    c = conn.cursor()

    # Create the articles table in the database
    c.execute('''CREATE TABLE IF NOT EXISTS articles_1
                (id INTEGER PRIMARY KEY, URL TEXT, headline TEXT, author TEXT, date TEXT)''')
    
     # Insert the article data into the database
    c.execute("INSERT or IGNORE INTO articles_1 (id, URL, headline, author, date) VALUES (?, ?, ?, ?, ?)",
            (index+1, link, headline, author, date))

    # Commit the changes to the database
    conn.commit()

    # Close the database connection
    conn.close()
```
## Wait for a specified amount of time before running the web scraper again to update the information
```
time.sleep(60)
