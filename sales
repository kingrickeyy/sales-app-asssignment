#!/usr/bin/python3
# -*- coding: UTF-8 -*-
import os
import sys
import json
from math import ceil

# Defining constants
TAX_BASIC = 10  # Basic tax on all goods except some 'specials'
TAX_IMPORT = 5  # Imported goods tax, applicable with no exemptions
TAX_EXEMPT = ['books', 'food', 'meds']  # 'Specials'
# Usage instructions
USAGE = '''
Usage:
    python3 sales_taxes.py [order_file]\n
Example:
    python3 sales_taxes.py data/order_1.txt
'''
# Order file format example
FFEX = '''
Order File format example:
1,LG0001
1,IG0001
2,LG0002
'''
# Catalog file example
CAT_EX = '''
Catalog file format example:
[
    {
    "sku": "LG0001",
    "name": "local item NAME",
    "category" : "beauty",
    "price": 9.99,
    "imported" : false
    },{
    "sku": "IG0002",
    "name": "imported item NAME",
    "category" : "books",
    "price": 15.29,
    "imported" : true
    }
]
'''


def main():
    """ Working with  """
    order = []
    if len(sys.argv) > 1:
        order = load_order(sys.argv[1])
    else:
        print("\nSales Taxes solution.\n")
        usage(0)
    if len(order) < 1:
        print("ERROR: Order is empty")
        usage(1)
    cart = ShoppingCart()
    for ent in order:
        cart.add(ent[0], ent[1])
    cart.print_receipt()
    sys.exit(0)


def usage(ec):
    """ Just to save few lines of code """
    print(USAGE)
    sys.exit(ec)


def orderfile_error(msg, line, cont):
    """ Returns error details if order is wrong """
    print(msg)
    print("Wrong line: {}, Content: '{}'".format(line, cont.strip('\n')))
    print(FFEX)
    sys.exit(1)


def tax_round(tax):
    """ Rounding rule for sales taxes """
    rnd = 20  # How many times 0.05 fits in 1.00? 20.
    # We will use round() function "floats behavior" to ensure that we have only 2 digits after decimal point
    return ceil(round(tax, 2) * rnd) / rnd


def load_order(order_file):
    """ Loading order file """
    orderlist = []
    with open(order_file, 'r') as order:
        data = order.readlines()
    for item in data:
        line = data.index(item) + 1
        try:
            quantity, sku = item.strip('\n').split(',')  # Remove newlines
        except ValueError:
            orderfile_error("ERROR: Order file format is broken.", line, item)
        quantity = int(quantity.strip(' '))  # Converting to int and stripping out whitespaces
        if quantity < 1:
            orderfile_error("ERROR: Quantity cannot be less than 1.", line, item)  # Returning an error and exitting
        sku = sku.strip(' ')  # Stripping out whitespaces
        entity = (quantity, sku)
        orderlist.append(entity)
    return orderlist


def check_catalog(catalog):
    """ Checking if catalog file has all needed keys """
    ckeys = ['sku', 'name', 'category', 'price', 'imported']
    for item in catalog:
        if not len([x for x in ckeys if x in item]) == len(ckeys):
            print("ERROR: Incomplete keys in catalog!")
            print("Wrong item: \n{}".format(item))
            print(CAT_EX)
            sys.exit(1)
    return True


# This method is placed here because we are using it in the class below
def load_catalog():
    """ Returning goods in list of dictionaries. """
    with open(os.path.join('data', 'catalog.json'), 'r') as cat:
        catalog = json.load(cat)
    if check_catalog(catalog):
        return catalog


class StorageItem():
    """ Class for our goods item """
    # Class variable. It saves some resources when we are getting it only once
    catalog = load_catalog()

    def __init__(self, sku):
        """ Get object attributes by sku """
        # I dare to say this is quite pythonic way to find if sku is present in catalog
        item = list(filter(lambda x: x.get('sku') == sku, StorageItem.catalog))
        if len(item) < 1:
            print("'{}'".format(sku))
            raise Exception('SKU Not Fonud.')
        self.name = item[0]['name']
        self.category = item[0]['category']
        self.price = item[0]['price']
        self.imported = item[0]['imported']
        self.tax = self.calc_tax()

    def calc_tax(self):
        """ Calculating tax """
        tax = 0
        if self.category not in TAX_EXEMPT:
            tax += TAX_BASIC
        if self.imported:
            tax += TAX_IMPORT
        # Convert tax to percents
        percent_tax = tax / 100
        # Total tax for price
        total_tax = self.price * percent_tax
        result_tax = tax_round(total_tax)
        return result_tax


class ShoppingCart():
    """ Here we can add our goods """
    def __init__(self):
        # This will be our shopping cart
        self.cart = []

    def add(self, quantity, sku):
        """ Adding items to our cart """
        # Creating a tuple (good for performance) with entity info.
        entity = (sku, quantity)
        self.cart.append(entity)

    def tax(self):
        """ Getting taxes for all items in cart """
        cart_tax = 0
        for ent in self.cart:
            item = StorageItem(ent[0])
            cart_tax += tax_round(item.tax * ent[1])
        return cart_tax

    def subtotal(self):
        """ Getting total price without taxes """
        cart_subtotal = 0
        for ent in self.cart:
            item = StorageItem(ent[0])
            cart_subtotal += item.price * ent[1]
        return cart_subtotal

    def total(self):
        """ Getting total price with taxes """
        cart_tax = self.tax()
        subtotal = self.subtotal()
        cart_total = cart_tax + subtotal
        return cart_tax, cart_total

    def print_receipt(self):
        """ Printing receipt """
        print("{0} SILLY MALL {0}".format("=" * 25))
        entities = []
        for ent in self.cart:
            item = StorageItem(ent[0])
            total = round((item.price + item.tax), 2)
            entities.append("{} {} at {:.2f}".format(ent[1], item.name, total))
        print("\n".join(entities))
        cart_taxes, cart_total = self.total()
        print("Sales Taxes: {0:.2f}".format(cart_taxes))
        print("Total: {0:.2f}".format(cart_total))
        print("{0} HAVE A NICE DAY! {0}".format("=" * 22))


if __name__ == "__main__":
    main()
