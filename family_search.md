# %%
# Import necessary libraries
import pandas as pd
import re
import matplotlib.pyplot as plt


# Load the CSV file with low_memory=False to avoid the mixed types warning
df = pd.read_csv(r'C:\BYUI\Family Search\combined (5).csv', low_memory=False)

# Step 1: Data Exploration and Initial Inspection
print("Initial Data Exploration:")
print(df[['245$f-Inclusive dates']].head(10))  # Preview the first 10 rows
print(f"Missing values in '245$f-Inclusive dates': {df['245$f-Inclusive dates'].isnull().sum()}")
print(f"Unique values in '245$f-Inclusive dates': {df['245$f-Inclusive dates'].unique()[:10]}")  # Preview unique values


# Step 2: Categorize Date Patterns
def categorize_date_pattern(date):
    if pd.isnull(date):
        return 'Missing'
    elif re.match(r'^\d{4}-\d{4}$', date):  # Match date ranges like "1990-1995"
        return 'Date Range'
    elif re.match(r'^\d{4}$', date):  # Match single years like "2005"
        return 'Single Year'
    else:
        return 'Other'  # Catch all for other formats like "1902 /", "Unknown", etc.

df['date_pattern'] = df['245$f-Inclusive dates'].apply(categorize_date_pattern)

# Count the occurrences of each date pattern
date_pattern_counts = df['date_pattern'].value_counts()

# Calculate the total count of date patterns
total_date_patterns = date_pattern_counts.sum()

# Create a DataFrame to store the count and percentage
date_pattern_df = pd.DataFrame(date_pattern_counts).reset_index()
date_pattern_df.columns = ['Date Pattern', 'Count']

# Calculate the percentage for each pattern
date_pattern_df['Overall Percentage'] = (date_pattern_df['Count'] / total_date_patterns) * 100
date_pattern_df['Overall Percentage'] = date_pattern_df['Overall Percentage'].apply(lambda x: f"{x:.2f}%")

# Display the final table
print("\nDate Pattern Counts with Overall Percentage:")
print(date_pattern_df.to_string(index=False))


# Step 3: Plotting Date Pattern Distribution
plt.figure(figsize=(10, 7))
bars = plt.bar(date_pattern_counts.index, date_pattern_counts.values, color=['blue', 'green', 'orange', 'red'])
plt.xlabel('Date Pattern Category')
plt.ylabel('Count of Records')
plt.title('Distribution of Date Patterns in 245$f-Inclusive dates Column')
plt.xticks(rotation=45)

# Add annotations
annotations = {
    'Missing': 'Examples:\nNaN\n...',
    'Date Range': 'Examples:\n"1549-1802",\n"1675-1811"',
    'Single Year': 'Examples:\n"1703",\n"1905"',
    'Other': 'Examples:\n"1585-1624, 1768-1804",\n"3/15/1900 1:00",\n"1624-1840 :"'
}
for i, bar in enumerate(bars):
    yval = bar.get_height()
    category = date_pattern_counts.index[i]  
    example_text = annotations[category]  
    plt.text(bar.get_x() + bar.get_width() / 2, yval + 5000, example_text, ha='center', va='bottom', fontsize=10)

plt.show()


######################################
# Data Cleaning and Special Character Analysis
######################################

# Step 4: Define Cleaning Functions

# Function to remove non-special characters
def remove_non_special_chars(df: pd.DataFrame, column_name: str) -> pd.DataFrame:
    pattern = r"[^@_!#$%^&*()<>?/\|}{~:]"  # Keep only the specified special characters
    df[column_name] = df[column_name].str.replace(pattern, "", regex=True)
    return df

# Function to remove numbers
def remove_numbers(df: pd.DataFrame, column_name: str) -> pd.DataFrame:
    pattern = r"\d"  # Remove digits
    df[column_name] = df[column_name].str.replace(pattern, "", regex=True)
    return df

# Apply cleaning functions to the '245$f-Inclusive dates' column
df['245$f-Cleaned'] = df['245$f-Inclusive dates']  # Create a copy for cleaning
df = remove_non_special_chars(df, '245$f-Cleaned')
df = remove_numbers(df, '245$f-Cleaned')

print("\nCleaned Data Example:")
print(df[['245$f-Inclusive dates', '245$f-Cleaned']].head(10))


# Step 5: Analyze Cleaned Data for Special Characters

# Function to find all special characters in the column
def find_all_special_characters(column_data):
    all_chars = set()
    for entry in column_data.dropna():
        special_chars = re.findall(r"[^a-zA-Z0-9\s]", entry)  # Find all special characters
        all_chars.update(special_chars)
    return all_chars

# Function to count special characters
def count_special_characters(column_data, special_chars):
    char_counts = {char: 0 for char in special_chars}
    total_count = 0
    for entry in column_data.dropna():
        for char in entry:
            if char in char_counts:
                char_counts[char] += 1
                total_count += 1
    return char_counts, total_count

# Function to calculate percentage of each character
def calculate_percentage(char_counts, total_count):
    char_data = []
    for char, count in char_counts.items():
        if count > 0:
            percentage = (count / total_count) * 100
            char_data.append([char, count, f"{percentage:.2f}%"])
    return pd.DataFrame(char_data, columns=['Character', 'Count', 'Overall Percentage'])

# Perform Special Character Analysis
cleaned_column_data = df['245$f-Cleaned']
all_special_chars = find_all_special_characters(cleaned_column_data)
char_counts, total_count = count_special_characters(cleaned_column_data, all_special_chars)
char_df = calculate_percentage(char_counts, total_count)

# Display the special character table
print("\nSpecial Character Analysis:")
print(char_df.to_string(index=False))

# Verify that percentages sum to 100%
total_percentage = sum([float(pct.strip('%')) for pct in char_df['Overall Percentage']])
print(f"\nTotal Percentage: {total_percentage:.2f}%")


# Step 1: Filter the DataFrame to focus only on rows where the 'date_pattern' is "Other"
df_other = df[df['date_pattern'] == 'Other'].copy()

# Step 2: Identify how many unique patterns exist in the "245$f-Inclusive dates" column for "Other" category
unique_patterns_in_other = df_other['245$f-Inclusive dates'].nunique()

# Step 3: Display the number of unique patterns in the "Other" category
print(f"\nTotal number of unique patterns in 'Other' category: {unique_patterns_in_other}")

# Step 4: Display a few examples of the unique patterns in the "Other" category
print("\nExamples of unique patterns in 'Other' category:")
print(df_other['245$f-Inclusive dates'].unique()[:10])  # Show the first 10 unique patterns


# Function to categorize 'Other' date patterns with additional cases
def categorize_other_pattern(date):
    if pd.isnull(date):
        return 'Missing'
    # Match date ranges with commas like "1585-1624, 1768-1804"
    elif re.match(r'^\d{4}-\d{4},\s?\d{4}-\d{4}$', date):
        return 'Date Range with Commas'
    # Match datetime formats like "3/15/1900 1:00"
    elif re.match(r'^\d{1,2}/\d{1,2}/\d{4}\s\d{1,2}:\d{2}', date):
        return 'Datetime Format'
    # Match date ranges with colons or periods at the end like "1624-1840:", "1872-1923."
    elif re.match(r'^\d{4}-\d{4}\s?[.:]$', date):
        return 'Date Range with End Punctuation'
    # Match date ranges with semicolons and indexes like "1800-1860; index,1802-1881"
    elif re.match(r'^\d{4}-\d{4};\s?index,\d{4}-\d{4}$', date):
        return 'Date Range with Semicolon and Index'
    # Match single years with commas like "1829, 1839"
    elif re.match(r'^\d{4},\s?\d{4}$', date):
        return 'Single Years with Comma'
    # Match incomplete date formats like "1803-1902 /" or "1969 /"
    elif re.match(r'^\d{4}-\d{4}\s?/$', date) or re.match(r'^\d{4}\s?/$', date):
        return 'Incomplete Date Range'
    # Match single years followed by periods like "1872-1923."
    elif re.match(r'^\d{4}-\d{4}\s?[.]$', date) or re.match(r'^\d{4}\s?[.]$', date):
        return 'Single Year with End Punctuation'
    # Match dates with multiple dashes like "1772--2007"
    elif re.match(r'^\d{4}--\d{4}$', date):
        return 'Date Range with Multiple Dashes'
    # Match improper date formatting with no delimiters like "18801890", "19071922"
    elif re.match(r'^\d{8}$', date):
        return 'Dates with Missing Delimiters'
    # Match shorter improper formats like "18251838" or "900-1834"
    elif re.match(r'^\d{3,4}-\d{3,4}$', date):
        return 'Improper Date Range'
    else:
        return 'Other Unspecified Pattern'




# %%
# Step 1: Filter the DataFrame to focus only on rows where the original 'date_pattern' is "Other"
df_other = df[df['date_pattern'] == 'Other'].copy()

# Step 2: Apply the function to create a new column 'Other_pattern' for the detected patterns
df_other['Other_pattern'] = df_other['245$f-Inclusive dates'].apply(categorize_other_pattern)

# Step 3: Count the occurrences of each pattern in the "Other_pattern" column
other_pattern_counts = df_other['Other_pattern'].value_counts()

# Step 4: Display the pattern counts
print("\nPattern Counts in 'Other' Category:")
print(other_pattern_counts)

# Step 5: View specific examples of rows with the "Date Range with Commas" pattern
date_range_with_commas = df_other[df_other['Other_pattern'] == 'Date Range with Commas']
print("\nExamples of 'Date Range with Commas' pattern:")
print(date_range_with_commas[['245$f-Inclusive dates']].head())


# %%