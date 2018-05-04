# Final Project: Kick-Help API
#### Isaiah Discipulo
## Overview
For this project I created a Python API to predict the success of a Kickstarter campaign. This API uses a neural network to estimate the probability of success for a given campaign. This project was concieved at the Sunhacks hackathon, in collaboration with Luis Gomez and Sam Rondinelli. I have since spent the last month improving the API as a solo developer.
## Data
To train my model, I used a dataset of Kickstarter projects, posted on Kaggle by user Kemical. The dataset included over 380,000 projects with several parameters, including Kickstarter category, duration of fundraising, USD goal, and project success. Below is a sample of the raw dataset.
```
1000002330,The Songs of Adelaide & Abullah,Poetry,Publishing,GBP,2015-10-09 11:36:00,1000,2015-08-11 12:12:28,0,failed,0,GB,0,,,,
1000004038,Where is Hank?,Narrative Film,Film & Video,USD,2013-02-26 00:20:50,45000,2013-01-12 00:20:50,220,failed,3,US,220,,,,
1000007540,ToshiCapital Rekordz Needs Help to Complete Album,Music,Music,USD,2012-04-16 04:24:11,5000,2012-03-17 03:24:11,1,failed,1,US,1,,,,
1000011046,Community Film Project: The Art of Neighborhood Filmmaking,Film & Video,Film & Video,USD,2015-08-29 01:00:00,19500,2015-07-04 08:35:03,1283,canceled,14,US,1283,,,,
1000014025,Monarch Espresso Bar,Restaurants,Food,USD,2016-04-01 13:38:27,50000,2016-02-26 13:38:27,52375,successful,224,US,52375,,,,
1000023410,Support Solar Roasted Coffee & Green Energy!  SolarCoffee.co,Food,Food,USD,2014-12-21 18:30:44,1000,2014-12-01 18:30:44,1205,successful,16,US,1205,,,,
1000030581,Chaser Strips. Our Strips make Shots their B*tch!,Drinks,Food,USD,2016-03-17 19:05:12,25000,2016-02-01 20:05:12,453,failed,40,US,453,,,,
1000034518,SPIN - Premium Retractable In-Ear Headphones with Mic,Product Design,Design,USD,2014-05-29 18:14:43,125000,2014-04-24 18:14:43,8233,canceled,58,US,8233,,,,
```
Although I have not confirmed it, I would guess that this dataset was generated by scraping Kickstarter. Unfortunately, certain scraping techniques can be less reliable than say, a Kickstarter API, and the dataset had many flaws. I wrote several functions in Python to clean the data and transfrom it to a useable state. The `clean` function is the top-level command to clean the data. It reads in a csv file, extracting from each line the project name, duration of fundraising, USD goal, and project success. Some of the categories in the dataset are no longer valid Kickstarter categories, so those projects are thrown out. Additionally, I was only interested in projects that completed their fundraising run and either succeded or failed. Many projects in the dataset were cancelled or suspended prior to completing their fundraising run, so those projects were also thrown out. To prepare the data to be fed into the neural network, the projects have their categories enumerated with the `get_category` function and their duration cast from the datetime format to a double with the `get_duraction` function. The data is then restructred with the `make_project` function and stored as a csv. The clean dataset has approximately 78,000 projects. Below is a sample of the clean dataset.
```
toshicapital rekordz needs help to complete album,4,2595600.0,5000,0
support solar roasted coffee & green energy!  solarcoffee.co,7,1728000.0,1000,1
the cottage market,12,2592000.0,5000,0
g-spot place for gamers to connect with eachother & go pro!,0,3884400.0,200000,0
survival rings,1,2592000.0,2500,0
mike corey's darkness & light album,4,1296000.0,250,1
boco tea,7,2592000.0,5000,0
cmuk. shoes: take on life feet first.,5,3024000.0,20000,1
```
## Model
To model the success of Kickstarter projects I chose to use a neural network. My motivations behind this were twofold: I had no initial hypothesis about the relationship between the features and success and I have been wanting to learn the TensorFlow and Keras libraries for a while. As mentioned above, I chose to use Kickstarter category, duration of fundraising, and USD goal as the features. My output is binary: did the project succeed or fail at being funded. Prior to training the model, I loaded in the clean dataset and scaled the features using the `scale` function to the domain `[0,1]`. The doubles are then cast to numpy's `float32` and reformatted to arrays. After researching online, I found the reccommended activation function for keras models with binary output is a sigmoid. In the process of training my model, I wanted to determine the optimal tuning parameters. Unfortunately, with a time complexity of `O(n^c)` it was unrealistic to change more than one parameter. I chose to fix the batch size at `50`, the epochs at `1000` and the learning rate at `0.05`. I then varied the number of nodes in a single layer neural network from `1` to `101`. The error was computed using root mean squared and the loss was computed using keras' `binary_accuracy` function. Below is a plot of the model accuracy vs. the number of the nodes.  

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

Since I was looking for the highest accuracy, I inverted the results so that my plot matched the typical U-shaped plots from class. As you can see, the optimal number of nodes for my data and parameters is `11`. I then retrained the model with the optimal parameters, but set the epochs to `10,000` to allow for more learning. The final accuracy of the model is approximately `0.65`. This model was then stored for later use.
## Predict
To predict the success of a given Kickstarter project, I first had to get the project data. To accomplish this I used a helper function, `get_page`. Given a search term, the function returns the data for the Kickstarter project most closely matching the term. I chose to use closest match as opposed to a URL because the format of individual Kickstarter pages can vary greatly. Variables like the browser or the author's chosen format can change where data is located on the page and makes it difficult to scrape reliably. A page that never changes however is the Kickstarter search results page. This page alway returns a JSON object of a particular format. The `get_page` function queries this JSON object to extract the data, which is recast to numpy's `float32` and reformatted to an array. I then apply the same `scale` function from earlier so that the prediction inputs are analagous to the model training data. The `predict` function then loads the model I trained earlier and returns the probability of project success.
## UI
Originally, this project had a front-end built as a RESTful web application, which communicated with a Python Flask server to predict project success. As I have developped the project on my own, I have moved away from the RESTful front end, as this is not my forte nor the focus of this particular project. In my api file, `kickhelp.py`, I have written a simple command line client to allow for the prediction of project success.
## Challenges and Improvements
The first improvement I would like to make would be the integration of undirected machine learning to quantify the project descriptions. I attempted to implement a rudimentary version using the `word2vec` library; however, I was unable to succesfully integrate this into my model. I would guess that the compellingness of a description affects whether people pledge or not, so I am likely losing important information by not including this feature in my model. The second improvement would be updating the front end, so that there is a functional RESTful web application to access the project through.
## Code
Below is the code I wrote for this project.
```
import csv
from datetime import datetime
import os
import keras
from keras import Model
from keras.models import Sequential
from keras.layers import Dense, Activation, Dropout
import h5py
import numpy as np
import requests
import json

CATEGORY_MAX = 14 # taken from previous run on data
DURATION_MAX = 7948800 # taken from previous run on data
GOAL_MAX = 100000000 # taken from previous run on data
CATEGORIES = ['games',
			'design',
			'technology',
			'film & video',
			'music',
			'fashion',
			'publishing',
			'food',
			'art',
			'comics',
			'theater',
			'photography',
			'crafts',
			'dance',
			'journalism']

#####################################
#          CLEAN FUNCTIONS          #
#####################################

# clean data set
def clean(infile, outfile):
	# init
	raw_project_count = 0
	clean_project_count = 0
	clean_successful_count = 0
	clean_failed_count = 0
	# read from csv
	print('Cleaning Project List...')
	with open(infile, 'r', encoding='latin-1') as csvfile:
		reader = csv.DictReader(csvfile)
		data = [r for r in reader]
		x = []
		# clean
		for p in data:
			try:
				name = p['name'].lower()
				category = p['category'].lower()
				duration = get_duration(p['launched'], p['deadline'])
				goal = p['goal']
				outcome = p['state']
				if outcome == 'successful' and category in CATEGORIES:
					category = get_category(category)
					new_p = make_project(name, category, duration, goal, 1)
					x.append(new_p)
				elif outcome == 'failed' and category in CATEGORIES:
					category = get_category(category)
					new_p = make_project(name, category, duration, goal, 0)
					x.append(new_p)
			except:
				pass
	# write to csv
	print('Writing Clean List...')
	with open(outfile, 'w', encoding='latin-1') as csvfile:
		keys = list(x[0].keys())
		writer = csv.DictWriter(csvfile, fieldnames=keys, lineterminator='\n')
		writer.writeheader()
		for p in x:
			writer.writerow(p)	
	# summary
	raw_project_count = len(data)
	clean_project_count = len(x)
	clean_successful_count = sum(p['outcome'] == 1 for p in x)
	clean_failed_count = sum(p['outcome'] == 0 for p in x)
	print('Raw Project Count:', raw_project_count)
	print('Clean Project Count:', clean_project_count)
	print('Clean Succesful Count:', clean_successful_count)
	print('Clean Failed Count:', clean_failed_count)

# category to num
def  get_category(category):
	return CATEGORIES.index(category)

# datetime to num
def get_duration(launched, deadline):
    t1 = datetime.strptime(launched[0:18], '%Y-%m-%d %H:%M:%S')
    t2 = datetime.strptime(deadline[0:18], '%Y-%m-%d %H:%M:%S')
    t3 = t2 - t1
    return(str(t3.total_seconds()))

# project structure
def make_project(name, category, duration, goal, outcome):
	data = {'name': '',
			'category': '',
			'duration': '',
			'goal': '',
			'outcome': ''}
	data['category'] = category
	data['duration'] = duration
	data['goal'] = goal
	data['name'] = name
	data['outcome'] = outcome
	return data

#####################################
#          TRAIN FUNCTIONS          #
#####################################

# load data from clean
def load_data(infile):
	x_train = []
	y_train = []
	# load
	with open(infile, 'r', encoding='latin-1') as csvfile:
		reader = csv.DictReader(csvfile)
		data = [r for r in reader]
	v_stack = np.empty((0, 3), dtype='float32')
	# format
	for p in data:
		category = np.float32(p['category'])
		duration = np.float32(p['duration'])
		goal = np.float32(p['goal'])
		features = [category, duration, goal]
		h_stack = np.hstack(features)
		v_stack = np.vstack([v_stack, h_stack])
		y_train.append(p['outcome'])
	x_train = np.array(v_stack)
	y_train = np.array(y_train)
	return x_train, y_train

# scale x's
def scale(x_train):
	category_max = x_train[:,0].max()
	duration_max = x_train[:,1].max()
	goal_max = x_train[:,2].max()
	x_train[:,0] = [n / category_max for n in x_train[:,0]]
	x_train[:,1] = [n / duration_max for n in x_train[:,1]]
	x_train[:,2] = [n / goal_max for n in x_train[:,2]]
	return x_train

# train keras model
def model_train(x_train, y_train, batch_size, epochs, lr, nodes, save, outfile='model.h5'):
	batch_size = batch_size
	epochs = epochs
	lr = lr
	input_dim = (x_train.shape[1])
	model = Sequential()
	model.add(Dense(nodes, input_dim=input_dim, activation='sigmoid'))
	model.add(Dense(1, activation='sigmoid'))
	opt = keras.optimizers.rmsprop(lr=lr)
	model.compile(optimizer=opt, loss='binary_crossentropy', metrics=['binary_accuracy'])
	metrics = model.fit(x=x_train, y=y_train,
			batch_size=batch_size,
			epochs=epochs,
			verbose=1,
			)
	if save == 1:
		model.save(outfile)
	return metrics.history['binary_accuracy'][0]

# find optimal parameters
def model_validate(x_train, y_train, batch_size, epochs, lr, min_nodes, max_nodes, step, outfile='model_accuracy.txt'):
	# options
	batch_size = batch_size # 50
	epochs = epochs # 1000
	lr = lr # 0.05
	nodes = np.arange(min_nodes, max_nodes, step) # [1, 11, ... 101]
	x = []
	# validate
	for i in nodes:
		print("Model With", i, "Nodes...")
		accuracy = model_train(x_train, y_train, batch_size, epochs, lr, i, 0)
		x.append(accuracy)
	# save accuracy
	np.savetxt(outfile, x, fmt='%f')

	max_accuracy = max(x)
	opt_nodes = x.index(max_accuracy) * step + min_nodes
	return opt_nodes, max_accuracy

#####################################
#        PREDICT FUNCTIONS          #
#####################################

# scrape kickstarter
def get_page(title):
	data = {'name': '',
			'category': '',
			'duration': '',
			'goal': ''}
	request_string = 'https://www.kickstarter.com/projects/search.json?search=&term={}'.format('-'.join(title.split(' ')))
	page = requests.get(request_string)
	response = page.json()
	if response['total_hits'] == 0:
		return -1
	else:
		project = response['projects'][0]
		data['name'] = project['name']
		data['category'] = get_category(project['category']['slug'].split('/')[0])
		data['duration'] = np.float32(project['deadline'] - project['launched_at'])
		data['goal'] = np.float32(project['goal'])
		return data 

# predict success
def predict(title, model='model.h5'):
	model = keras.models.load_model(model)
	data = get_page(title)
	if data == -1:
		print('Project Could Not Be Found')
	else:
		category = data['category']
		duration = data['duration']
		goal = data['goal']
		x = np.array([category/CATEGORY_MAX, duration/DURATION_MAX, goal/GOAL_MAX])
		x.shape = (1, len(x))
		result = float(model.predict(x))
		print('Probability of Success For', data['name'],':', result)

if __name__ == '__main__':
	# test clean functions
	#print()
	#clean('raw_projects_1.csv', 'clean_projects_1.csv')

	# test model functions
	#print()
	#batch_size = 50
	#epochs = 1
	#lr = 0.05
	#min_nodes = 1
	#max_nodes = 3
	#step = 1
	#x_train, y_train = load_data('clean_projects_1.csv')
	#x_train = scale(x_train)
	#opt_nodes, accuracy = model_validate(x_train, y_train, batch_size, epochs, lr, min_nodes, max_nodes, step, 'test_accuracy.txt')
	#model_train(x_train, y_train, batch_size, epochs, lr, opt_nodes, 1, 'test_model.h5')

	# test predict functions
	print()
	url = input('Enter Project Title: ').lower()
	print('Predicting...\n')
	predict(url)
  ```
