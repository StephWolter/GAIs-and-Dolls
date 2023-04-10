# An Analysis of Book Publishers and Bestseller Lists, by GAIs-and-Dolls

### Summary

This is a data analysis of book publishers using the New York Times (NYT) Books API. Specifically, we evaluate the performance of book publishers in terms of their representation on NYT bestseller lists, in order to identify trends in the industry as a whole. The following steps are taken to perform the analysis:

1. In Python, make a series of API calls to the NYT Books API, request the responses in json, populate a pandas dataframe, and save the dataframe as a CSV file.
2. In Python, create an initial visualization of book publishers' performance in terms of their relative representation on NYT bestseller lists. 
3. In Excel and Tableau, refine the initial visualization by integrating a) historical information, and b) corporate ownership information. 

### Step 1. Retrieving and Storing NYT API Data

The NYT Books API gives developers information about the NYT Bestseller lists. After creating an account and generating a personal API key, developers gain access to https://api.nytimes.com/svc/books/v3/lists. The List Data service, lists/{date}/{name}, "returns the books on the best sellers list for the specified date and list name." See https://developer.nytimes.com/docs/books-product/1/overview for reference.     

In order to obtain the List Data across multiple lists and multiple dates, we will make a series of API calls. The code that does this is contained in the file, 'BooksPy/Get_NYT_Bestsellers.ipynb'. This file imports required libraries, defines a datetime function, makes API calls, populates a dataframe, and saves the dataframe as a CSV file.   

##### Dependencies and imports

Get_NYT_Bestsellers.ipynb imports the following libraries: 

1. Datetime, to enable datetime objects used in the API requests;
2. Requests, for the API requests themselves;
3. Pandas, to store the API responses in a dataframe; and
4. Time, to insert pauses between API requests

Get_NYT_Bestsellers.ipynb also imports an API key from the api_keys.py file. A .gitignore file exists in the repo to prevent the private API key from being shared with the public.

##### Create a list of dates and a list of bestseller lists

The NYT Book Review is published on Sundays. To generate a list of Sundays for our queries, we define a function, getPreviousSunday. This function takes a date object as an input, and returns as an output the date object of a Sunday from the prior month. With getPreviousSunday defined, we then create and populate the variable, search_dates, which is a list of date objects.  

Next, we create a populate the variable, nyt_lists, which is a list of the names of NYT bestseller lists.

##### Loop through the dates and bestseller lists

Once we have a list of dates and a list of bestseller lists, we iterate through the lists to make a series of calls to the List Data service. Before doing so, we initialize an empty list called book_data, into which we will append the API responses. Then we create two for-loops, one for search_dates and one for nyt_lists. With each iteration through these for-loops, we refresh the List Data url and use requests.get(url).json() to store the API response in json. The relevant information is then pulled from the response variable, based on the json structure defined by NYT at: https://developer.nytimes.com/docs/books-product/1/routes/lists/%7Bdate%7D/%7Blist%7D.json/get. This information includes:

1. The (non-encoded) name of the bestseller list;
2. The published date of the bestseller list; and
3. An array containing information about each book on the bestseller list. 

For each book in the array, the response provides the following information:

1. Title;
2. Author;
3. Publisher;
4. Weeks on the corresponding bestseller list; and
5. Identifiers (the 10-digit and 13-digit ISBN codes).

The above information is stored in the book_data variable using the .append() function. 

##### Error handling and API limitations 

During the process of repeated API calls, there may be failures. We therefore add error handling, which makes a note of the failed request and proceeds to the next iteration instead of halting execution altogether. 

Notwithstanding the error handling, we noted certain limitations with the NYT API. For one, we noticed a surge in failed requests after approximately 5 iterations. We discovered that such failures can be avoided by adding a ~20 second delay between successive API requests. This is accomplished using the sleep() function from the time library. Although the delay prevents failed API requests, it dramatically increases the amount of time required for code execution. Furthermore, NYT policy sets a rate limit for each user, prohibiting bulk data requests. Further work is needed to find alternative solutions to these limitations.     

##### Store data

To store the data obtained from the NYT API, we convert book_data into a pandas dataframe using pd.DataFrame(). We then use pd.to_csv() to save the dataframe as a CSV file ('NYT_Bestsellers.csv')

### Step 2. Initial Data Visualization

'BooksPy/Book_Performance.ipynb' contains code that performs an initial visualization of the data stored in NYT_Bestsellers.csv. Specifically, we take the latest NYT bestseller lists and compute the total weeks on the bestseller lists for each publisher.

##### Text Wrapping Function

Before we load the data and create the visualizations, we define a function called wrap_labels. This function was written by Ted Petrou and shared at: https://medium.com/dunder-data/automatically-wrap-graph-labels-in-matplotlib-and-seaborn-a48740bc9ce.

The wrap_labels function, which relies on the textwrap library, takes a matplotlib axis and a user-specified number of characters as inputs. The function converts the axis label strings from a continuous display to a wrapped display, with the wrapping taking place after nth character, where n is defined by the user.

Since some publisher names are long strings with many characters, the labels overlap when displayed in a Matplotlib plot. Although the overlap can be eliminated by rotating the labels, displaying the labels in this way is cumbersome. With the wrap_labels function, we will be able to display the publisher names from left-to-right, making our charts much more natural to read for the audience.

##### Use pandas to isolate the current NYT bestseller lists

Using pandas, we call DataFrame.read_csv() to populate a dataframe from 'NYT_Bestsellers.csv'. We then filter out any observations from lists published before the latest list, which is the 9 April 2023 publication as of the date of this analysis. We then separate the dataframe into genre-specific dataframes as follows:

1. Adult fiction, which comprises the following bestseller lists: Combined Print and E-Book Fiction, Hardcover Fiction, and Trade Fiction Paperback.
2. Young adult fiction and children, which comprises the following bestseller lists: Young Adult Hardcover, Series Books, and Picture Books.
3. Nonfiction, which comprises the following bestseller lists: Combined Print and E-Book Nonfiction, Hardcover Nonfiction, and Advice How-To and Miscellaneous. 

##### Create bar charts

Using Matplotlib's DataFrame.plot() function, we create bar charts for each genre-specific dataframe. Other Matplotlib functions, (e.g., plt.title), and the wrap_labels function are used to label the chart and make aesthetic changes. The plt.savefig() function is used to save the charts as a png file.

### Step 3. Integration of Time-Series and Corporate Ownership Information

In order to integrate/append publisher name (typically subsidiaries) to their parent company, we initially created a comprehensive list that identifies publishers and imprints to their parent companies. To explore and gauge the effiencies of different programs and programming languages, we employed three methods of data appending on the same data. The first was Power Query in Excel, VLookup in Excel, and lastly Python. All these methods included adding a new conditional column titled 'Parent Group', which based on the value in the 'Publisher' column would give an output of the corresponding parent publishing company.

##### Inserting a conditional column through Excel Power Query

Fairly straightforward, this method only requires retrieving data from our 'NYT_Bestsellers.csv'(our data source), inserting a conditional column with the new name 'Parent Group', and inputting the data using the provided 'else if' options. Once the data is loaded, it immediately populates the relevant cells in the newly created column.

##### Filling a column through Excel Visual Basic for Applications (VBA)

For this process, VBA If statement is used to determine the output based on the values of a certain cell. A for loop is used to ensure the rule applies throughout the column.

##### Inserting a conditional column through Pandas Python in Jupyter Notebook

Following the tyical imports, including that of the relevant cvs file, we load the file as a dataframe. Using the pandas .map() method, we create a dictionary to set values to append in the new column destination.

Throughout the data preprocessing, there proved to be no significant difference in the use of either program, the only extensive and tedious work encountered during the data integration process was in establishing a comprehensive list detailing the relationship between the numerous publishers and their corporate owners. This list ("PublishingHouses.xlsx) was necessary in order to create a way to recognize established patterns found in the generated Best Seller lists.

##### Data Vizualization

Using excel, python pandas and Tableau, a series of bar graphs, pie charts, and sankey diagrams were created to display trends in the data. This includes higlighting the dominance of the top 5 publishing companies through .value_counts() in python or pivot charting in Excel, and showcasing the minimal changes in the dominance of publishers throughout the years.