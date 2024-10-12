Certainly! Below is a comprehensive guide to creating, setting up, and running a **Streamlit-based Redbus Route Data Scraper** using **Selenium**, **BeautifulSoup**, **Pandas**, and **MySQL**. This guide includes the complete code, step-by-step instructions, and explanations to help you get started and successfully execute the project.

---

## **Table of Contents**

1. [Prerequisites](#prerequisites)
2. [Step 1: Install Required Software](#step-1-install-required-software)
3. [Step 2: Install Python Packages](#step-2-install-python-packages)
4. [Step 3: Set Up MySQL Database](#step-3-set-up-mysql-database)
5. [Step 4: Download and Configure ChromeDriver](#step-4-download-and-configure-chromedriver)
6. [Step 5: Create the Python Script (`app.py`)](#step-5-create-the-python-script-apppy)
7. [Step 6: Run the Streamlit Application](#step-6-run-the-streamlit-application)
8. [Step 7: Using the Application](#step-7-using-the-application)
9. [Troubleshooting](#troubleshooting)
10. [Complete Code](#complete-code)

---

## **Prerequisites**

Before proceeding, ensure you have the following:

- **Python** installed on your system (preferably Python 3.7 or higher).
- **MySQL** installed and running.
- **Google Chrome** installed (version must match ChromeDriver).
- **Basic knowledge** of Python and command-line operations.

---

## **Step 1: Install Required Software**

### **1.1 Install Python**

If you haven't installed Python yet:

- **Windows/Mac/Linux**: Download the installer from the [official Python website](https://www.python.org/downloads/) and follow the installation instructions.

### **1.2 Install MySQL**

- **Download MySQL**: Visit the [MySQL Downloads](https://dev.mysql.com/downloads/installer/) page and download the MySQL Community Server.
- **Installation**: Follow the installer instructions. During installation:
  - Set a **root password** (remember it for later).
  - Optionally, create a new user with specific privileges.

---

## **Step 2: Install Python Packages**

Open your **Command Prompt (Windows)** or **Terminal (macOS/Linux)** and run the following commands to install the necessary Python packages:

```bash
pip install selenium beautifulsoup4 pandas streamlit mysql-connector-python
```

**Packages Overview:**

- **Selenium**: Automates web browser interaction.
- **BeautifulSoup**: Parses HTML and XML documents.
- **Pandas**: Data manipulation and analysis.
- **Streamlit**: Builds interactive web applications.
- **MySQL Connector**: Connects Python to MySQL databases.

---

## **Step 3: Set Up MySQL Database**

### **3.1 Start MySQL Server**

Ensure your MySQL server is running. You can start it via the **MySQL Workbench** or using command-line tools.

### **3.2 Create Database and Table**

1. **Access MySQL**:

   Open the **MySQL Command Line Client** or **MySQL Workbench** and log in using your credentials.

2. **Create Database**:

   ```sql
   CREATE DATABASE redbus;
   ```

3. **Use Database**:

   ```sql
   USE redbus;
   ```

4. **Create Table**:

   ```sql
   CREATE TABLE bus_routes (
       id INT AUTO_INCREMENT PRIMARY KEY,
       source VARCHAR(255),
       destination VARCHAR(255),
       date DATE,
       bus_name VARCHAR(255),
       departure_time VARCHAR(50),
       arrival_time VARCHAR(50),
       duration VARCHAR(50),
       price DECIMAL(10,2)
   );
   ```

   **Explanation of Columns:**

   - **id**: Primary key.
   - **source**: Starting location.
   - **destination**: Ending location.
   - **date**: Travel date.
   - **bus_name**: Name of the bus operator.
   - **departure_time**: Departure time.
   - **arrival_time**: Arrival time.
   - **duration**: Journey duration.
   - **price**: Ticket price.

---

## **Step 4: Download and Configure ChromeDriver**

**Selenium** requires a browser driver to interact with browsers. We'll use **ChromeDriver** for Google Chrome.

### **4.1 Check Your Chrome Version**

1. Open **Google Chrome**.
2. Navigate to `chrome://settings/help`.
3. Note the **Chrome version** (e.g., 115.0.5790.170).

### **4.2 Download Matching ChromeDriver**

1. Visit the [ChromeDriver Downloads](https://sites.google.com/a/chromium.org/chromedriver/downloads) page.
2. Download the **ChromeDriver** version that matches your Chrome browser version.
3. **Extract** the downloaded file.

### **4.3 Place ChromeDriver in a Known Directory**

For example, place it in `C:\webdrivers\chromedriver.exe` (Windows) or `/usr/local/bin/chromedriver` (macOS/Linux).

**Note**: Ensure the **ChromeDriver** executable has the necessary permissions to execute.

---

## **Step 5: Create the Python Script (`app.py`)**

Create a Python script named `app.py` in your project directory (e.g., `C:\Users\YourUsername\YourProject\app.py`) with the following complete code:

```python
import streamlit as st
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.options import Options
from bs4 import BeautifulSoup
import pandas as pd
import mysql.connector
import time

# Configure Streamlit page
st.set_page_config(page_title="Redbus Route Data Scraper", layout="wide")

# Establish the MySQL connection
def get_db_connection():
    return mysql.connector.connect(
        host="127.0.0.1",       # Typically "localhost" or "127.0.0.1" for local MySQL server
        user="root",            # Your MySQL username
        password="1234",        # Your MySQL password
        database="redbus"       # Your database name
    )

# Function to store data in MySQL
def store_data_in_mysql(data, source, destination, date):
    try:
        mydb = get_db_connection()
        cursor = mydb.cursor()
        for entry in data:
            cursor.execute(
                """
                INSERT INTO bus_routes (source, destination, date, bus_name, departure_time, arrival_time, duration, price)
                VALUES (%s, %s, %s, %s, %s, %s, %s, %s)
                """,
                (
                    source,
                    destination,
                    date,
                    entry['bus_name'],
                    entry['departure_time'],
                    entry['arrival_time'],
                    entry['duration'],
                    entry['price']
                )
            )
        mydb.commit()
        cursor.close()
        mydb.close()
        st.success("Data successfully stored in MySQL database.")
    except mysql.connector.Error as err:
        st.error(f"Error: {err}")

# Function to scrape Redbus data
def scrape_redbus_data(source, destination, date):
    # Configure Chrome options
    chrome_options = Options()
    chrome_options.add_argument("--headless")  # Run Chrome in headless mode
    chrome_options.add_argument("--no-sandbox")
    chrome_options.add_argument("--disable-dev-shm-usage")
    
    # Path to ChromeDriver
    chrome_driver_path = 'C:/webdrivers/chromedriver.exe'  # Update this path
    
    # Initialize WebDriver
    service = Service(executable_path=chrome_driver_path)
    driver = webdriver.Chrome(service=service, options=chrome_options)
    
    # Navigate to Redbus
    driver.get("https://www.redbus.in/")
    
    try:
        # Input source
        source_input = driver.find_element(By.ID, "src")
        source_input.clear()
        source_input.send_keys(source)
        time.sleep(2)  # Wait for autocomplete suggestions
        source_input.send_keys("\n")  # Select the first suggestion
        
        # Input destination
        dest_input = driver.find_element(By.ID, "dest")
        dest_input.clear()
        dest_input.send_keys(destination)
        time.sleep(2)  # Wait for autocomplete suggestions
        dest_input.send_keys("\n")  # Select the first suggestion
        
        # Input date
        date_input = driver.find_element(By.ID, "onward_cal")
        date_input.click()
        # Select the date from the calendar
        date_str = date.strftime("%d %b %Y")
        date_element = driver.find_element(By.XPATH, f"//td[@class='wd day' and text()='{date.day}']")
        date_element.click()
        
        # Click search button
        search_button = driver.find_element(By.ID, "search_btn")
        search_button.click()
        
        # Wait for results to load
        time.sleep(5)
        
        # Scroll down to load all results
        driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
        time.sleep(2)
        
        # Get page source and parse with BeautifulSoup
        soup = BeautifulSoup(driver.page_source, 'html.parser')
        
        # Extract bus details
        buses = soup.find_all('div', class_='bus-item')  # Update this selector based on actual Redbus HTML structure
        
        data = []
        for bus in buses:
            try:
                bus_name = bus.find('div', class_='travels').text.strip()
                departure_time = bus.find('div', class_='dp-time').text.strip()
                arrival_time = bus.find('div', class_='bp-time').text.strip()
                duration = bus.find('div', class_='dur').text.strip()
                price = bus.find('div', class_='fare').text.strip().replace('₹', '').replace(',', '')
                
                data.append({
                    'bus_name': bus_name,
                    'departure_time': departure_time,
                    'arrival_time': arrival_time,
                    'duration': duration,
                    'price': float(price)
                })
            except AttributeError:
                # Skip if any attribute is missing
                continue
        
        # Close the WebDriver
        driver.quit()
        
        return data
    except Exception as e:
        driver.quit()
        st.error(f"An error occurred during scraping: {e}")
        return []

# Streamlit App
def main():
    st.title("🚌 Redbus Route Data Scraper")
    st.markdown("Enter your travel details to scrape bus routes from Redbus.")
    
    # Input fields
    col1, col2, col3 = st.columns(3)
    
    with col1:
        source = st.text_input("**Source**", "Bangalore")
    
    with col2:
        destination = st.text_input("**Destination**", "Chennai")
    
    with col3:
        date = st.date_input("**Date**")
    
    if st.button("**Search**"):
        if not source or not destination or not date:
            st.warning("Please fill in all fields.")
        else:
            with st.spinner("Scraping data..."):
                data = scrape_redbus_data(source, destination, date)
            
            if data:
                df = pd.DataFrame(data)
                st.success(f"Found {len(df)} bus routes.")
                st.dataframe(df)
                store_data_in_mysql(data, source, destination, date)
            else:
                st.warning("No data found.")

if __name__ == "__main__":
    main()
```

### **Code Explanation**

1. **Imports**: Import necessary libraries for web scraping, data handling, database operations, and building the web app.
2. **MySQL Connection**: Defines a function to establish a connection to the MySQL database.
3. **Data Storage**: Function to insert scraped data into the MySQL database.
4. **Scraping Function**: Uses Selenium to navigate Redbus, input search parameters, and scrape bus details.
5. **Streamlit Interface**: Provides a user interface to input travel details and display the scraped data.

---

## **Step 6: Run the Streamlit Application**

### **6.1 Open Terminal/Command Prompt**

Navigate to your project directory where `app.py` is saved.

```bash
cd C:\Users\YourUsername\YourProject
```

### **6.2 Run the Streamlit App**

Execute the following command to run the Streamlit application:

```bash
streamlit run app.py
```

**Expected Output:**

After running the command, you should see an output similar to:

```plaintext
  You can now view your Streamlit app in your browser.

  Network URL: http://192.168.1.100:8501
  External URL: http://your-ip-address:8501
```

### **6.3 Open the App in Browser**

- **Local Access**: Open your web browser and navigate to `http://localhost:8501`.
- **Remote Access**: If accessing remotely, use the **External URL** provided in the terminal.

---

## **Step 7: Using the Application**

1. **Input Travel Details**:
   - **Source**: Enter the starting location (e.g., "Bangalore").
   - **Destination**: Enter the ending location (e.g., "Chennai").
   - **Date**: Select the travel date using the date picker.

2. **Search for Buses**:
   - Click the **"Search"** button.
   - The app will display a spinner indicating that data is being scraped.

3. **View Results**:
   - Once scraping is complete, the app will display the number of bus routes found.
   - The data will be presented in a table format.
   - The same data will be stored in the MySQL database under the `bus_routes` table.

---

## **Troubleshooting**

### **1. ChromeDriver Errors**

- **Version Mismatch**: Ensure that the ChromeDriver version matches your installed Chrome browser version.
- **Path Issues**: Verify that the `chrome_driver_path` in `app.py` points to the correct location of `chromedriver.exe`.

### **2. MySQL Connection Errors**

- **Credentials**: Ensure that the MySQL `user`, `password`, `host`, and `database` details in the script are correct.
- **Database and Table**: Confirm that the `redbus` database and `bus_routes` table exist.

### **3. Selenium Issues**

- **Selectors**: The HTML structure of Redbus may change over time. If scraping fails, inspect the Redbus website to update the element selectors in the script.
- **Dynamic Content**: Redbus may load content dynamically. Adjust `time.sleep()` durations or use Selenium's **Explicit Waits** for better reliability.

### **4. Streamlit App Not Loading**

- **Port Issues**: If port `8501` is in use, specify a different port:

  ```bash
  streamlit run app.py --server.port 8502
  ```

- **Firewall/Antivirus**: Ensure that your firewall or antivirus isn't blocking Streamlit.

---

## **Complete Code**

Below is the complete `app.py` script for your reference:

```python
import streamlit as st
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.options import Options
from bs4 import BeautifulSoup
import pandas as pd
import mysql.connector
import time

# Configure Streamlit page
st.set_page_config(page_title="Redbus Route Data Scraper", layout="wide")

# Establish the MySQL connection
def get_db_connection():
    return mysql.connector.connect(
        host="127.0.0.1",       # Typically "localhost" or "127.0.0.1" for local MySQL server
        user="root",            # Your MySQL username
        password="1234",        # Your MySQL password
        database="redbus"       # Your database name
    )

# Function to store data in MySQL
def store_data_in_mysql(data, source, destination, date):
    try:
        mydb = get_db_connection()
        cursor = mydb.cursor()
        for entry in data:
            cursor.execute(
                """
                INSERT INTO bus_routes (source, destination, date, bus_name, departure_time, arrival_time, duration, price)
                VALUES (%s, %s, %s, %s, %s, %s, %s, %s)
                """,
                (
                    source,
                    destination,
                    date,
                    entry['bus_name'],
                    entry['departure_time'],
                    entry['arrival_time'],
                    entry['duration'],
                    entry['price']
                )
            )
        mydb.commit()
        cursor.close()
        mydb.close()
        st.success("Data successfully stored in MySQL database.")
    except mysql.connector.Error as err:
        st.error(f"Error: {err}")

# Function to scrape Redbus data
def scrape_redbus_data(source, destination, date):
    # Configure Chrome options
    chrome_options = Options()
    chrome_options.add_argument("--headless")  # Run Chrome in headless mode
    chrome_options.add_argument("--no-sandbox")
    chrome_options.add_argument("--disable-dev-shm-usage")
    
    # Path to ChromeDriver
    chrome_driver_path = 'C:/webdrivers/chromedriver.exe'  # Update this path
    
    # Initialize WebDriver
    service = Service(executable_path=chrome_driver_path)
    driver = webdriver.Chrome(service=service, options=chrome_options)
    
    # Navigate to Redbus
    driver.get("https://www.redbus.in/")
    
    try:
        # Input source
        source_input = driver.find_element(By.ID, "src")
        source_input.clear()
        source_input.send_keys(source)
        time.sleep(2)  # Wait for autocomplete suggestions
        source_input.send_keys("\n")  # Select the first suggestion
        
        # Input destination
        dest_input = driver.find_element(By.ID, "dest")
        dest_input.clear()
        dest_input.send_keys(destination)
        time.sleep(2)  # Wait for autocomplete suggestions
        dest_input.send_keys("\n")  # Select the first suggestion
        
        # Input date
        date_input = driver.find_element(By.ID, "onward_cal")
        date_input.click()
        # Select the date from the calendar
        date_str = date.strftime("%d %b %Y")
        date_element = driver.find_element(By.XPATH, f"//td[@class='wd day' and text()='{date.day}']")
        date_element.click()
        
        # Click search button
        search_button = driver.find_element(By.ID, "search_btn")
        search_button.click()
        
        # Wait for results to load
        time.sleep(5)
        
        # Scroll down to load all results
        driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
        time.sleep(2)
        
        # Get page source and parse with BeautifulSoup
        soup = BeautifulSoup(driver.page_source, 'html.parser')
        
        # Extract bus details
        buses = soup.find_all('div', class_='bus-item')  # Update this selector based on actual Redbus HTML structure
        
        data = []
        for bus in buses:
            try:
                bus_name = bus.find('div', class_='travels').text.strip()
                departure_time = bus.find('div', class_='dp-time').text.strip()
                arrival_time = bus.find('div', class_='bp-time').text.strip()
                duration = bus.find('div', class_='dur').text.strip()
                price = bus.find('div', class_='fare').text.strip().replace('₹', '').replace(',', '')
                
                data.append({
                    'bus_name': bus_name,
                    'departure_time': departure_time,
                    'arrival_time': arrival_time,
                    'duration': duration,
                    'price': float(price)
                })
            except AttributeError:
                # Skip if any attribute is missing
                continue
        
        # Close the WebDriver
        driver.quit()
        
        return data
    except Exception as e:
        driver.quit()
        st.error(f"An error occurred during scraping: {e}")
        return []

# Streamlit App
def main():
    st.title("🚌 Redbus Route Data Scraper")
    st.markdown("Enter your travel details to scrape bus routes from Redbus.")
    
    # Input fields
    col1, col2, col3 = st.columns(3)
    
    with col1:
        source = st.text_input("**Source**", "Bangalore")
    
    with col2:
        destination = st.text_input("**Destination**", "Chennai")
    
    with col3:
        date = st.date_input("**Date**")
    
    if st.button("**Search**"):
        if not source or not destination or not date:
            st.warning("Please fill in all fields.")
        else:
            with st.spinner("Scraping data..."):
                data = scrape_redbus_data(source, destination, date)
            
            if data:
                df = pd.DataFrame(data)
                st.success(f"Found {len(df)} bus routes.")
                st.dataframe(df)
                store_data_in_mysql(data, source, destination, date)
            else:
                st.warning("No data found.")

if __name__ == "__main__":
    main()
```

---

## **Additional Notes**

1. **Selectors**: The CSS selectors used in the scraping function (`find_element(By.ID, "src")`, etc.) are based on the current structure of the Redbus website. If Redbus updates their website, these selectors might break. Inspect the website and update the selectors accordingly.

2. **Dynamic Content**: Redbus may use JavaScript to load content dynamically. Selenium handles JavaScript rendering, but ensure adequate wait times (`time.sleep()`) or use Selenium's **Explicit Waits** for better reliability.

3. **Headless Mode**: The script runs Chrome in headless mode, meaning it operates without a GUI. If you face issues, try running without headless mode by removing or commenting out the `--headless` argument to see the browser actions.

4. **Error Handling**: The script includes basic error handling. For production use, consider enhancing error handling and logging mechanisms.

5. **Security**: Avoid hardcoding sensitive information like database passwords in your scripts. Consider using environment variables or configuration files with proper security measures.

---

## **Conclusion**

You now have a complete setup for a Streamlit application that scrapes bus route data from Redbus and stores it in a MySQL database. By following the steps outlined above, you should be able to run the application, input your travel details, and retrieve and store relevant bus information.

Feel free to customize and expand upon this foundation to better suit your specific requirements. If you encounter any issues or have further questions, don't hesitate to ask!
