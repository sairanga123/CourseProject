# Improved ExpertSearch

# Project Source Code

The project's source code is available under the ExpertSearch folder.

# Project Documentation

The project's detailed report is available as:
![Project Report - Improved ExpertSearch.pdf](Project Report - Improved ExpertSearch.pdf)

The project's tutorial presentation video is available as:
![Video Presentation - Improved ExpertSearch.mp4](Video Presentation - Improved ExpertSearch.mp4)

# PLEASE READ "Project Report - Improved ExpertSearch.pdf" FOR THE REPORT
# PLEASE VIEW "Video Presentation - Improved ExpertSearch.mp4" FOR THE PRESENTATION AND DEMO

# Improved ExpertSearch System

(Team ZMS)

Zacharia Rupp (zrupp2@illinois.edu)

Mriganka Sarma (ms76@illinois.edu)

Sai Ranganathan (sr50@illinois.edu)


# **Introduction**

The ExpertSearch system is a system to search faculties who are experts in certain research areas or topics from university websites crawled from the web. The goal of our project is to improve this ExpertSearch system in a few ways, including automating the scraping process, adding topic mining for finding faculty research topics, and improving the UI to give improved visualizations and query refinement options.

# **Functional Overview**

The improved ExpertSearch system enables the following functionalities:

- Automatic scraping of websites to identify faculty directory webpages and non-directory webpages
- Automatic scraping of the classified faculty directory webpages to further identify faculty bio webpages and non-bio webpages
- Automatic scraping of faculty bio webpages to generate one bio document per faculty and adding to the compiled bios
- Topic Mining from the compiled bios to extract research topics of the faculties
- Display top-5 research topics associated with each retrieved faculty
- Improved email extraction for each faculty
- Refine search query using any of the topics from the displayed topic cloud
- Prepopulate email content when clicked on a faculty&#39;s email address

The **Automated Scraper** improves the faculty bio generation process from a vast collection of websites.

The **Topic Miner** adds more structure to the unstructured faculty website data retrieved from a query.

The **enhanced UI** enables succinct visualization of the structured faculty results and provides shortcuts for additional search filters and faculty connection.

Together, these new features improve the utility of the ExpertSearch system to the user.

# **Implementation Details**

## **Automated scraping process**

The Automated Scraper takes a set of known University websites and top 500 Alexa websites as input, performs a series of operations to classify the directory URLs and then to classify the faculty homepages. The automated scraper then scrapes the classified faculty homepages to generate faculty bio documents and adds the bios to the collection.

### **Inputs:**

- University websites
- Top-500 Alexa websites

### **Outputs:**

- Corpus of classified Faculty Directory URLs _**(classified\_dir\_urls.cor)**_
- Corpus of classified Faculty Bio URLs _**(classified\_faculty\_urls.cor)**_
- Documents of bios for each faculty generated by scraping the classified Faculty Bio URLs _**(e.g. 6530.txt)**_

### **Deliverables:**

- Automated Scraper _**(auto\_scraper.py)**_
- Data Handler _**(data\_handler.py)**_
- Scraper _**(scraper.py)**_
- Text Classifier _**(text\_classifier.py)**_

### **Component design / Code workflow:**

### **Automated Scraper (auto\_scraper.py)**

The Automated Scraper module automates the whole flow of generating the faculty bios from the input mixture of &quot;positive&quot; and &quot;negative&quot; URLs in the following sequence of steps:

- Uses the data handler (data\_handler.py) to prepare a train and test set of Faculty Directory URLs
- Uses the scraper (scraper.py) to scrape these URLs to prepare the train and test corpus
- Uses the text classifier (text\_classifier.py) to build and train a Doc2Vec model on the documents in the train corpus of directory contents
- Uses the text classifier to predict the category of the test URLs as &quot;Directory&quot; or &quot;Non-Directory&quot;
- Saves the classified directory URLs to a file **(classified\_dir\_urls.cor)**
- Uses the data handler to prepare a train and test set of Faculty Bio URLs
- Uses the scraper to scrape these URLs to prepare the train and test corpus
- Uses the text classifier to build and train a Doc2Vec model on the documents in the train corpus of faculty bios
- Uses the text classifier to predict the category of the test URLs as &quot;Faculty&quot; or &quot;Non-Faculty&quot;
- Saves the classified bio URLs to a file **(classified\_faculty\_urls.cor)**
- Uses the scraper to scrape the faculty bios from the classified bio URLs
- Generates one document per faculty bio (e.g. **6530.txt** ) and saves under **ExpertSearch/data/compiled\_bios**

The auto\_scraper.py module is the entry point for the complete automatic scraping and bio generation task.

### **Directory URL Classification**

###

First, let&#39;s explain the Directory URL Classification task with the help of the modules.

#### **Data Handler (data\_handler.py)**

The Data Handler module first takes the University websites and Alexa websites as input, mixes them and partitions them into test and train URLs. Then uses the scraper module to extract the URL contents into test and train corpus.

Here&#39;s a detailed explanation of the approach used by the Data Handler module to prepare the URLs for the scraper.

  - Downloaded the known faculty directory URLs from the sign-up sheet for MP 2.1. These will serve as the &quot; **positive**&quot; examples.
    - About 900 URLs were obtained from the sign-up sheet data, which was partitioned into **500 for training** and **400 for test data**.
  - Collected top URLs from Alexa. These will serve as the &quot; **negative**&quot; examples.
    - Collected the global top-50 pages of Alexa.
    - Collected the top-50 pages for 14 different countries. Manually verified that the pages are in English.
    - This gave 750 &quot;negative&quot; URLs.
  - When the **AutoScraper** is launched, it invokes the **DataHandler** which mixes and partitions the above URLs into train and test URLs as follows:
    - Training URLs
  - Converts the MP 2.1 sign-up sheet from csv to a file containing only the directory URLs. Performs any cleanup as necessary and labels them as &quot;directory&quot;.
  - Combines the top-50 Alexa URLs for 10 countries and labels them as &quot;alexa\_dir&quot;. Uses these 500 pages for training.
  - Mixes the 500 Faculty Directory training URLs with the 500 Alexa training URLs. Removes duplicates if any. This gives 734 URLs as the final training URLs.
  - The training URLs are saved in the file **train\_urls.cor**.
    - Test URLs
  - Combines the top-50 Alexa URLs for 5 countries. Uses these 250 pages for testing.
  - Mixes the 400 Faculty Directory test URLs with the 250 Alexa test URLs. Remove duplicates if any. This gives 548 URLs as the final test URLs.
  - The test URLs are saved in the file **test\_urls.cor**.

#### **Scraper (scraper.py)**

The Scraper module scrapes the contents from the above train and test URLs and prepares the train and test corpus for the classification task.

The scraper does the following:

  - Gets the contents of each URL as text.
  - Performs clean-up of non-ascii characters from the content.
  - Performs other clean-ups such as substituting newlines, tabs, multiple whitespaces into single whitespace.
  - Substitutes contents such as &quot;403 Forbidden&quot;, &quot;404 Not found&quot;, etc. with &quot;Error: Content Not Found&quot;.
  - Writes contents of each training URL as a single line of space separated words to the training corpus (&quot; **train\_dataset.cor**&quot;).
  - Similarly, writes contents of each test URL as a single line of space separated words to the test corpus (&quot; **test\_dataset.cor**&quot;).

#### **Text Classifier (text\_classifier.py)**

The Text Classifier module uses the train and test dataset from above step to classify the Faculty Directory URLs.

The classification module does the following:

  - Uses **gensim** to build a Doc2Vec model for feature vector representation of each document.
  - Uses the train\_dataset.cor to build the vocabulary and train the model.
  - Saves the model so that it can be reloaded while running next time on the same dataset.
  - Uses **LogisticRegression** as the classifier from scikit-learn module.
  - Uses **LogisticRegression** to predict the categories of the test URLs given the test dataset.

### **Faculty URL classification**

Next, let&#39;s look at the Faculty URL Classification task.

#### **Data Handler (data\_handler.py)**

The Data Handler module now takes the existing project&#39;s known Faculty Bio URLs and mixes with some Alexa URLs to prepare the train dataset. Uses the classified Faculty Directory URLs from the above step to extract potential Faculty Bio URLs to prepare the test dataset. Then uses the scraper module to extract the URL contents into bio test and bio train corpus.

The following approach was used to prepare the dataset:

  - Training URLs
- Use the top 1000 URLs from the currently existing Faculty Bio URLs in the ExpertSearch project as the &quot; **positive**&quot; train URLs.
- Use 250 URLs from the Alexa test URLs set as the &quot; **negative**&quot; train URLs.
- Combine them.
- Tag the faculty bio URLs as &quot;faculty&quot; and save to the file &quot; **train\_bio\_urls.cor**&quot;.
- Tag the Alexa URLs as &quot;alexa\_faculty&quot; and save to the same file.
- This will be the final file with all the train URLs.
  - Test URLs
- Use the Classified Faculty Directory URLs obtained from the Directory URL Classification task above.
- Use the scraper to find all potential faculty bio URLs from each of these Directory URLs.
- Save all these potential faculty bio URLs to the file &quot; **test\_bio\_urls.cor**&quot;.
- This will be the final file with all the test URLs.

#### **Scraper (scraper.py)**

Since the ExpertSearch project already contains the faculty bios as documents, the contents of the top 1000 faculty bios **(0.txt … 999.txt)** are copied to the train corpus file (&quot; **train\_bio\_dataset.cor**&quot;).

Then the scraper does the following:

  - Scrapes the URLs from line no. 1000 till the end from the file **train\_bio\_urls.cor** and appends to the train corpus (&quot; **train\_bio\_dataset.cor**&quot;).
  - Scrapes the contents of the test URLs (i.e. potential faculty bio URLs) from the **test\_bio\_urls.cor** file above and adds those contents to the test corpus (&quot; **test\_bio\_dataset.cor**&quot;).

#### **Text Classifier (text\_classifier.py)**

The classification module does the following:

  - Uses **gensim** to build a **Doc2Vec** model for feature vector representation of each document.
  - Uses the **train\_bio\_dataset.cor** to build the vocabulary and train the model.
  - Saves the model so that it can be reloaded while running next time on the same dataset.
  - Uses **LogisticRegression** as the classifier from scikit-learn module.
  - Uses **LogisticRegression** to predict the categories (bio or non-bio) of the test URLs given the test dataset.

Finally, the Scraper module scrapes these classified bio URLs and saves the contents of each bio URL to a new file under _ExpertSearch/data/compiled\_bios_.

## **Topic Mining**

### **Inputs:**

- Generated Faculty Bio documents from the AutoScraper _**(e.g. 6530.txt)**_

### **Outputs:**

- Trained topic model _**(lda\_mallet\_model)**_
- Bag-of-words representation of corpus to be used with miner.py _**(corpus\_dictionary)**_
- Text representation of corpus _**(lda\_corpus)**_

### **Deliverables:**

- Python script to create topic model and retrieve top-10 terms associated with query topic _**(miner.py)**_

### **Component design / Code workflow:**

### **Topic Miner (miner.py)**

The topic miner pulls a topic distribution from a document already mined if the model was trained on the document, otherwise it uses gensim and mallet to create a model from the entire corpus. The process is described below:

#### **Corpus preparation**

  - Read in compiled bios as strings
  - Filter the string representation of each bio to:
- Remove stop words
- Extract HTML tags and elements
- Strip non-alphanumeric characters
- Strip numbers
- Strip words that exist in lists of terms extracted from the bios
- Strip words that exist in a manually defined list of words that were creating incoherent topic clusters ( **unwanted\_words.txt** )
- Remove words shorter than four characters
- Split all words into a list of tokens
  - Create list of documents which is comprised of lists of tokens for each document as described above
  - Append bigrams and trigrams to each token list for each document
  - Create a gensim dictionary from the above documents
  - Create a bag-of-word representation of our documents: this will be our corpus.

#### **Model creation**

Model creation required a good deal of manual work to ensure that the term clusters were understandable. The process consisted of a lot of trial and error, using the steps below:

  - Create a general model with **gensim.models.ldamodel.LdaModel** class with 10 models
  - Visually inspect term clusters to ensure they were meaningful
  - If the above criteria were not satisfactory:
- Tweak corpus construction
  - After the above criteria was deemed satisfactory:

- Using **gensim.models.wrappers.LdaMallet** with the mallet library, I:
  1. Varied number of topics to create new model
  2. Assessed coherence of each model with varying number of topics
  3. Manually inspected output of models with high coherence, looking for term clusters that made intuitive sense and that appeared distinct given knowledge of the separate domains
  4. Chose the best model according to above criteria and saved it and the created dictionary for query inference


#### **Term extraction**

With a model and a dictionary, I wrote a method that can infer the topic of a given query and fetch the top-10 terms associated with that topic, a method that can infer the topic of a single document and fetch the top-10 terms associated with that document&#39;s topic, and a method that can infer the topics of multiple documents and fetch the top-10 terms associated with each document&#39;s topic. These terms will eventually be pushed to the user to help them potentially refine their query.

## **Improved Email Extraction**

### **Inputs:**

- Generated Faculty Bio documents from the AutoScraper _**(e.g. 6530.txt)**_

### **Outputs:**

- Extracted emails for the faculties

### **Deliverables:**

- Updated email extractor _**(extract-email.py)**_

### **Component design / Code workflow:**

#### **Regex Improvement:**

- There are certain edge cases that we had noticed in some of the email web pages where the format was different than the traditional email formatting
- Added more regex matches in order to match with these edge cases such as ex. rohini[@]buffalo[DOT]edu

## **UI Improvements**

### **Inputs:**

- Query terms in the search box
- Topics mined by the topic miner

### **Outputs:**

- Updated UI showing top-5 research topic per faculty
- Updated UI showing topic cloud
- Clickable topic terms for query refinement
- Prepopulated email template on-click email icon

### **Deliverables:**

- Updated server endpoints _**(**_ **server** _**.py)**_
- Updated UI _**(index.js)**_

### **Component design / Code workflow:**

#### **Info Button:**

- Information button is created at the top of each of the retrieved faculty.
- When the button is clicked there a table pops up that appears below the selected retrieved faculty
- The table will contain additional information regarding the research topics that the faculty does
- When one of the info buttons associated with a faculty is clicked the other one will close, and the new one will open.

#### **Top 5 Topics Display:**

- Display the top 5 topics from the preview for each of the faculty.
- Display these topics in a table format when the information button is clicked
- Underneath each topic is a &#39;Learn More&#39; button which when clicked leads you to a page talking more about the topic in detail from the web.
- Clicking on the &quot;Add to Query&quot; button will refine the current search query to include this topic.

#### **Email Automation:**

- Email comes pre-populated with a set subject and body.
- The body talks about one of the research topics that were extracted from the top 5 topics and how the user would like to connect with the faculty regarding research in this topic.

# **Usage Details**

The modified project has been tested on Mac and Windows with Python 2.7. Here are the setup instructions for each of these platforms.

## **Setup Guide (Mac)**

### **Repo setup**

Run the following command on a terminal to clone the github repository.

**$ git clone https://github.com/sairanga123/CourseProject.git**

### **Project environment setup**

The project has been tested on python 2.7. Please setup a python 2.7 environment for running the project. Creating an environment from Anaconda will make many common packages available. So a quick way to start would be to setup a python 2.7 environment from Anaconda.

May need to install many or all of the following python packages depending on what packages the python environment already has.

  - gunicorn=19.10.0
  - flask=1.1.2
  - metapy=0.2.13
  - requests=2.25.0
  - pytoml=0.1.21
  - gensim=3.8.3
  - nltk=3.4
  - bs4=0.0.1
  - lxml=4.6.2
  - numpy=1.16.6
  - sklearn=0.0

## **Setup Guide (Windows)**

Windows is currently not supported. If you want to build in Windows, use Windows Subsystem for Linux and follow the steps above.

## **Usage Guide**

### **Running the Automated Scraper**

Run the AutoScraper to generate the bio documents. This step has been already performed and the generated bio documents have already been added to the **ExpertSearch/data/compiled\_bios** folder. Here are the instructions for running the AutoScraper if it needs to be run again with additional input.

To run the automated scraper, first go the ExpertSearch/AutoScraper directory.

![](RackMultipart20201214-4-ybwwhv_html_1ad0d364bddc0951.png)

Then the Automated Scraper can be invoked as follows:

![](RackMultipart20201214-4-ybwwhv_html_26877c84163faee6.png)

**-d** option specifies to generate/regenerate the train and test dataset.

If **-d** switch is used:

- If the dataset already exists, it will be regenerated
- If the dataset doesn&#39;t yet exist, it will be generated

If **-d** switch is not used:

- If the dataset already exists, the existing dataset will be used in the subsequent flow
- If the dataset doesn&#39;t yet exist, it will be generated even if -d switch is not used

**-t** option specifies to train/retrain the Doc2Vec model on the train dataset.

If **-t** switch is used:

- If a trained and saved model already exists, the model will be retrained and saved again
- If a trained and saved model doesn&#39;t yet exist, it will be trained and saved

If **-t** switch is not used:

- If a trained and saved model already exists, the saved model will be loaded and used for inference in the subsequent flow
- If a trained and saved model doesn&#39;t yet exist, it will be trained and saved even if -t switch is not used

### **Running the Topic Miner**

The steps for creating the topic model are documented in **ExpertSearch/data/expertsearch/LDATopicModeling.ipynb**. The model construction is not something that can necessarily be automated because relying on perplexity and coherence scores alone often results in topics that don&#39;t make any meaningful sense to a human.

Once the topic model is constructed and saved, miner.py allows the server to load the model and make inferences.

### **Running the Backend Server**

Once, the topic model has been built, we can start the server.

To start the server, go to the ExpertSearch folder.

Then run the following command:

**$ gunicorn server:app -b 127.0.0.1:8095**

### **Running Faculty Search from the UI**

Now, launch a web browser and type the following URL:

**localhost:8095**

## **Example Use Cases:**

### **Use Case 1 – Basic use to search faculties**

Let&#39;s assume that we want to find the faculties that are working on &quot; **text mining**&quot;.

Then we&#39;d go to the UI and enter our search string as &quot;text mining&quot;.

The existing system would retrieve the top ranked faculty results working on &quot;text mining&quot;.

### **Use Case 2 – Find research interests of the faculty**

Now, the improved system will also provide an info button which will bring up an additional table of information for each faculty. This table shows the top 5 research topics the faculty is associated with.

### **Use Case 3 – Find faculties working on similar topics as the faculty from the initial search results**

We now maybe interested in learning more about who are the faculties that are working on any of these research topics. We can quickly search for all the faculties working on this new research topic by simply clicking on the &quot;Add to Query&quot; button for that research topic. This will automatically modify our search query by including that new research topic without having to type it in the search box. The retrieved faculty results will show the list of faculties working on that research topic.

### **Use Case 4 – Connecting to faculty**

Another way we could use the system is to click on the email icon to send an email to the faculty&#39;s email address. While we may be at a loss of words for that first email, the system will provide a pre-populated template email which will automatically address the faculty&#39;s name and also include reference to the faculty&#39;s research area. This will make connecting to an expert faculty just one click away.

# **Contributions**

| **Item**| **Sub-items** | **Contributor** |
| --- | --- | --- |
| Automated Scraping |- Automated Scraper to automate the complete process<br></br->- Data Handler to prepare the datasets for the text classification tasks<br></br->- Scraper to scrape the URLs<br></br->- Text Classifier to classify directory and bio URLs<br></br->- Function to generate bio documents and add to compiled bios| Mriganka Sarma |
| Topic Mining |- Topic model<br>- Function to return top-10 words associated with query topic| Zacharia Rupp|
| Improved Email Extraction |- Added regular expressions to extract emails with atypical forms (e.g. person at place dot com)| Sai Ranganathan<br></brZacharia>Zacharia Rupp |
| Improved UI |- Display top 5 topics associated with each faculty member<br>- Display cloud of topics<br>- Pre-populate email field when clicked on email address<br>- Improve email extraction part 1. | Sai Ranganathan|



# Improved ExpertSearch Proposal

1. What are the names and NetIDs of all your team members? Who is the captain? The captain will have more administrative duties than team members.
    - Team Captain: Sai Ranganathan (sr50)
    - Mriganka Sarma (ms76)
    - Zacharia Rupp (zrupp2)
2. What system have you chosen? Which subtopic(s) under the system?
    - We have chosen to improve ExpertSearch.
3. Briefly describe the datasets, algorithms or techniques you plan to use
    - Datasets: 
        * Faculty dataset scraped from MP2.1 for positive examples.
        * Scrape of Alexa Top 500 Domains for negative examples.
    - Techniques:
        * Topic Mining
4. If you are adding a function, how will you demonstrate that it works as expected? If you are improving a function, how will you show your implementation actually works better?
    - Additional functionality:
        * Topic Mining
            - If it works, users will be able to refine queries by selecting topics from a topic cloud
            - Impact on ndcg@k
    - Improved functionality:
        * Email extraction
            - If it works, more faculty members will have email addresses associated with them.
    - Improved UI:
        * More granular query refinement
            - Top-k associated topics listed under individual faculty members
5. How will your code communicate with or utilize the system? It is also fine to build your own systems, just please state your plan clearly
    - Our code will build on the ExpertSearch code by: 
        * adding a topic mining function
        * improving email extraction
        * automating scraping process
        * improving UI
6. Which programming language do you plan to use?
    - Python
    - JavaScript
7. Please justify that the workload of your topic is at least 20*N hours, N being the total number of students in your team. You may list the main tasks to be completed, and the estimated time cost for each task.
    - Main tasks:
        * Automatic crawler to identify faculty directory pages (10+ hrs)
        * Automatic crawler to identify faculty webpage URLS (10+ hrs)
    - Improving functionality:
        * Email extraction (10+ hrs)
    - Adding functionality:
        * Topic mining (10+ hrs)
        * UI Improvements:
    - Query refinement options (10+ hrs)
        * Topic cloud from mined topics associated with retrieved faculty members.
        * Top-5 topics associated with faculty member (5+ hrs)
            - Displayed at the top of the bio excerpt
    - Prepopulated email content when a user clicks on a faculty member’s email address (5+ hrs)
        * E.g. 
            “Dear Faculty Name,        

            It’s a pleasure to have gone through some of your research articles. I’d like to connect with you for discussing some ideas in the Research Area.

            I hope to hear from you soon.”

