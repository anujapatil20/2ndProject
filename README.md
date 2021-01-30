# 2ndProject
Web scraping Project

import requests
from bs4 import Beautifulsoup
import pandas
import argparse
import connect

parser = argparse.ArgumentParser()
parser.add_argument("--page_num_max", help="Enter the number of pages to parse", type=int)
parser.add_argument("--dbname", help="Enter the number of pages to parse", type=int)
args = parser.parse_args()

oyo_url = "https://www.oyorooms.com/hotels-in-bangalore/?page="
page_num_max = args.page_num_max
scraped_info_list = []
connect.connect(args.dbname)

for page_num in range(1, page_num_max):
    req = requests.get(oyo_url + str(page_num))
    content = req.content

    soup = Beautifulsoup(content,"html.parser")

    all_hotels = soup.find_all("div", {"class":"hotelCradListing"})
    
    for hotel in all_hotels:
        hotel_dict = {}
        hotel_dict["name"] = hotel.find("h3", {"class": "listingHotelDescription_hotelName"}).text
        hotel_dict["address"] = hotel.find("span", {"itemprop": "streetAddress"}).text
        hotel_dict["price"] = hotel.find("span", {"class": "listingPrice__finalPrice"}).text

        #try ...except
        try:
            hotel_dict["name"] = hotel.find("span", {"class": "hotelRatin__ratingSummary"}).text
        except attributeError:
            pass
           
        parent_amenities_element = hotel.find("div", {"class": "amenityWrapper"})

        amenities_list = []
        for amenity in parent_amenities_element.find_all("div", {"class": "amenityWrapper__amenity"}):
            amenities_list.append(amenity.find("span", {"class": "d-body-sm"}).text.strip())
         
        hotel_dict["amenities"] = ','.join(amenities_list[:-1])
        
        scraped_info_list.append(hotel_dict)
        connect.insert_into_table(args.dbname, tuple(hotel_dict.values()))
        
        #print(hotel_name, hotel_address, hotel_price, hotel_rating)
        
dataFrame = pandas.DataFrame(scraped_info_list)
dataFrame.to_csv("Oyo.csv")    
