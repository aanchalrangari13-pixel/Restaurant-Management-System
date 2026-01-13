# Restaurant-Management-System
The Restaurant Management System is a software application designed to simplify and automate the daily operations of a restaurant. It helps manage orders, menu items, billing, and customer details efficiently, reducing manual work and improving overall productivity.
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import os

GST_RATE=0.05

#---MENU DATA---#
veg_menu={"Dosa":80,
          "Paneer Dosa":100,
          "Chole Bhature":150,
          "Veg Manchurian":160,
          "Veg Fried Rice":200,
          "Veg Biryani":220,
          "Paneer Tikka":230,
          "Chilli Paneer":250,
          "Paneer Butter Masala":260,}

nonveg_menu={"Egg Biryani":180,
         "Chicken Mnachurian":200,
         "Chicken Hot Garlic":220,
         "Chicken Fried Rice":240,
         "Chicken Biryani":280,
         "Butter Chicken":300,}

drink_menu={"Ice Tea":40,
            "Coffee":50,
            "Lemon Soda":50,
            "Hot Chocolate":70,
            "Cappuccino":100,
            "Cold Coffee":120}

#---COMBINED MENU FOR ORDER PROCESSING---#
MENU={**veg_menu, **nonveg_menu, **drink_menu}
order=[] 

#---FUNCTION---#
def display_menu():
    print("\n🍽️-------RESTAURANT MENU-------🍽️")
    
    print("\n🟢 VEG ITEMS")
    for item,price in veg_menu.items():
        print(f"{item:25} : Rs{price}")
        
    print("\n🔴 NON VEG ITEMS")
    for item,price in nonveg_menu.items():
        print(f"{item:25} : Rs{price}")
    
    print("\n🥂 DRINKS")
    for item,price in drink_menu.items():
        print(f"{item:25} : Rs{price}")
        
def take_order():
    while True:
        item=input("\nEnter item name (or 'done'): ").title()
        if item =="Done":
            break
        if item in MENU:
            qty=int(input("Enter Quantity: " ))
            total=MENU[item]*qty
            order.append([item,qty,MENU[item],total])
            print(f"✅ {qty} {item}(s) added")
        else:
            print("❌ Item not available")
            
def generate_bill(df):
    subtotal=np.sum(df["Total"])
    total_qty=np.sum(df["Quantity"])
    
    if total_qty>=10:
        discount_rate=0.15
    elif total_qty>=5:
        discount_rate=0.10
    else:
        discount_rate=0
        
    discount=subtotal*discount_rate
    gst=(subtotal-discount)*GST_RATE
    final_amount=subtotal-discount+gst
    
    print("\n📝------- BILL SUMMARY -------")
    print("Total Quantity :",total_qty)
    print("Subtotal       :Rs",subtotal)
    print("Discount       :Rs",round(discount,2))
    print("GST(5%)        :Rs",round(gst,2))
    print("Final Payable  :Rs",round(final_amount,2))
    
def save_order(df):
    if os.path.exists("restaurant_orders.csv"):
        old_df=pd.read_csv("restaurant_orders.csv")
        df=pd.concat([old_df, df],ignore_index=True)
       
    df.to_csv("restaurant_orders.csv",index=False)

def sales_analysis():
    df=pd.read_csv("restaurant_orders.csv")
    sales=df.groupby("Item")["Quantity"].sum()
    
    dish_items=list(veg_menu.keys())+list(nonveg_menu.keys())
    drinks_items=list(drink_menu.keys())
     
    dish_sales=sales[sales.index.isin(dish_items)]
    drink_sales=sales[sales.index.isin(drinks_items)]
    
    print("\n📊-------SALES ANALYSIS-------")
    if dish_sales.empty:
       print("⚠️ No dish sales data available")
    else:
        print("Most Selling Dish:",dish_sales.idxmax())
        print("Least Selling Dish:",dish_sales.idxmin())
    if drink_sales.empty:
        print("⚠️ No drink sales data available")
    else:    
        print("Most Selling Drinks:",drink_sales.idxmax())
        print("Least Selling Drinks:",drink_sales.idxmin())
    
#--- BAR GRAPH : DISH---#
    if not dish_sales.empty:
        plt.figure()
        plt.bar(dish_sales.index, dish_sales.values)
        plt.title("Dish Sales (Bar Graph)")
        plt.ylabel("Quantity Sold")
        plt.xticks(rotation=45)
        plt.tight_layout()
        plt.show()
        
# --- PIE CHART : DISH ---#
        plt.figure()
        plt.pie(dish_sales.values, labels=dish_sales.index,
                 autopct='%1.1f%%', startangle=140)
        plt.title("Dish Sales Distribution (Pie Chart)")
        plt.axis('equal')
        plt.show()
    
#---BAR GRAPH : DRINK ---#
    if not drink_sales.empty:
        plt.figure()
        plt.bar(drink_sales.index, drink_sales.values)
        plt.title("Drink Sales (Bar Graph)")
        plt.ylabel("Quantity Sold")
        plt.xticks(rotation=45)
        plt.tight_layout()
        plt.show()
        
# --- PIE CHART : DRINK ---#
        plt.figure()
        plt.pie(drink_sales.values, labels=drink_sales.index,
                 autopct='%1.1f%%', startangle=140)
        plt.title("Drink Sales Distribution (Pie Chart)")
        plt.axis('equal')
        plt.show()

    
#---MAIN PROGRAM---#
print("🍽️  Welcome to Eatza Restaurant Management System 🍽️")
display_menu()
take_order()
df=pd.DataFrame(order, columns=["Item","Quantity","Price","Total"])
print("\n📋 ORDER DETAILS")
print(df)

generate_bill(df)
save_order(df)

choice=input("\nView Sales Analysis? (yes/no): ").lower()
if choice=="yes":
    sales_analysis()
    
print("\n Thank You!!♥️ Hope You Vist Again" )