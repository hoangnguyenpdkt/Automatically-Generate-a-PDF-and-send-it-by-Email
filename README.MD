# Automatically Generate a PDF and send it by Email

## Introduction
You work for a company that sells second hand cars. Management wants to get a summary of the amounts of vehicles that have been sold at the end of every month. The company already has a web service which serves sales data at the end of every month but management wants an email to be sent out with an attached PDF so that data is more easily readable.
## What you’ll do
	Write a script that summarizes and processes sales data into different categories
	Generate a PDF using Python
	Automatically send a PDF by email


## Script file
Edit the script file
```
nano ~/scripts/cars.py
```
Add the folling lines:

```python
#!/usr/bin/env python3

import json
import locale
import sys
import emails
import os
import reports
from reportlab.platypus import Paragraph, Spacer, Table, Image
from reportlab.graphics.shapes import Drawing
from reportlab.graphics.charts.piecharts import Pie
from reportlab.platypus import SimpleDocTemplate
from reportlab.lib.styles import getSampleStyleSheet
from reportlab.lib import colors
from reportlab.lib.units import inch

def load_data(filename):
  """Loads the contents of filename as a JSON file."""
  with open(filename) as json_file:
    data = json.load(json_file)
  return data


def format_car(car):
  """Given a car dictionary, returns a nicely formatted name."""
  return "{} {} ({})".format(
      car["car_make"], car["car_model"], car["car_year"])


def process_data(data):
  """Analyzes the data, looking for maximums.

  Returns a list of lines that summarize the information.
  """
  max_revenue = {"revenue": 0}
  max_sales = {"total_sales":0}
  car_year = {}
  car_make = {}
  for item in data:
    # Calculate the revenue generated by this model (price * total_sales)
    # We need to convert the price from "$1234.56" to 1234.56
    item_price = locale.atof(item["price"].strip("$"))
    item_revenue = item["total_sales"] * item_price
    if item_revenue > max_revenue["revenue"]:
      item["revenue"] = item_revenue
      max_revenue = item
    # TODO: also handle max sales
    if item["total_sales"] > max_sales["total_sales"]:
        max_sales = item
    # TODO: also handle most popular car_year
    if item["car"]["car_year"] not in car_year:
        car_year[item["car"]["car_year"]] = item["total_sales"]
    else:
        car_year[item["car"]["car_year"]] += item["total_sales"]

    popular_car_year=(sorted(car_year.items(), key=lambda x: x[1], reverse=True)[0])
    # TODO: handle the total sales of each car made
    if item["car"]["car_make"] not in car_make:
        car_make[item["car"]["car_make"]] = item["total_sales"]
    else:
        car_make[item["car"]["car_make"]] += item["total_sales"]

  summary = [
    "The {} generated the most revenue: ${}".format(
      format_car(max_revenue["car"]), max_revenue["revenue"]),
    "The {} had the most Sales: {}".format(
        format_car(max_sales["car"]), max_sales["total_sales"]),
    "The most popular year was {} with: {} sales".format(popular_car_year[0], popular_car_year[1]),
  ]

  return summary, car_make


def cars_dict_to_table(car_data):
  """Turns the data in car_data into a list of lists."""
  table_data = [["ID", "Car", "Price", "Total Sales"]]
  for item in car_data:
    table_data.append([item["id"], format_car(item["car"]), item["price"], item["total_sales"]])
  return table_data


def main(argv):
  """Process the JSON data and generate a full report out of it."""
  data = load_data("car_sales.json")
  summary,car_make = process_data(data)
  #print(summary)
  # TODO: turn this into a PDF report
  # A paragraph which contains the most sales/most revenue/most popular year values
  
  text_summary = '<br/>\n'.join(summary)
  # A table which contains all the information parsed from the JSON file
  
  table_data = cars_dict_to_table(sorted(data, key = lambda i: i['total_sales'], reverse=True))
  # A pie chart for the total sales of each car made

  styles = getSampleStyleSheet()
  report_title = Paragraph("A Complete Summary of Monthly Car Sales", styles["h1"])
  report_text = Paragraph(text_summary, styles["h4"])
  table_style = [('GRID', (0,0), (-1,-1), 1, colors.black)]
  report_table = Table(data=table_data, style=table_style, hAlign="LEFT")
  report_pie = Pie(width=5*inch, height=5*inch)
  report_pie.data = []
  report_pie.labels = []
  for car in car_make:
      report_pie.data.append(car_make[car])
      report_pie.labels.append(car)

  report_chart = Drawing()
  report_chart.add(report_pie)

  report = SimpleDocTemplate("/tmp/report.pdf")
  report.build([report_title,report_text, report_table,report_chart])

  # TODO: send the PDF report as an email attachment
  sender = "automation@example.com"
  receiver = "{}@example.com".format(os.environ.get('USER'))
  subject = "Sales summary for last month"
  body = '\n'.join(summary)
  message = emails.generate(sender, receiver, subject, body, "/tmp/cars.pdf")
  emails.send(message)

if __name__ == "__main__":
  main(sys.argv)
```
Grant executable permission to the script file
```
sudo chmod o+wx ~/scripts/cars.py
```
Now, run the file.
```
./scripts/cars.py
```
Lab-3: Automating Real-World Tasks, Google IT Automation with Python Professional Certificate
