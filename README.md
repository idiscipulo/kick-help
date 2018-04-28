# Final Project: Kick-Help API
#### Isaiah Discipulo
## Overview
For this project I created a Python API to predict the success of a Kickstarter campaign, called Kick-Help. Kick-Help uses a neural network to estimate the probability of success for a given campaign. This project was concieved at the Sunhacks hackathon, in collaboration with Luis Gomez and Sam Rondinelli. I have since spent the last month improving the API as a solo developer.
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
Although I have not confirmed it, I would guess that this dataset was generated by scraping Kickstarter. Unfortunately, certain scraping techniques can be less reliable than say, a Kickstarter API, and the dataset had many flaws. I wrote several functions in Python to clean the data and transfrom it to a useable state. The `clean` function is the top-level command to clean the data. It reads in a csv file, extracting from each line the project name, duration of fundraising, USD goal, and project success. Some of the categories in the dataset are no longer valid Kickstarter categories, so those projects are thrown out. Additionally, I was only interested in projects that completed their fundraising run and either succeded or failed. Many projects in the dataset were cancelled or suspended prior to completing their fundraising run, so those projects were thrown out. To prepare the data to be fed into the neural network, the projects had their categories enumerated with the `get_category` function and their duration cast from the datetime format to a double. The data was then restructred with `make_project` and stored as a csv. Below is a sample of the clean dataset.
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
To model the success of Kickstarter projects I chose to use a neural network. My motivations behind this were twofold: I had no initial hypothesis about the relationship between the features and success and I have been wanting to learn the TensorFlow and Keras libraries for a while. For the features, I chose to use Kickstarter category, duration of fundraising, and USD goal. My output is binary: did the project succeed or fail at being funded. For the transformation function I chose to use the sigmoid function and set my output threshold at `0.5`. To train and validate my models I chose to fix the batch size at `50`, the epochs at `1000` and the learning rate at `0.05`, then varied the number of nodes in a single layers neural network. I tested nodes in the range `[1,101]`, using root mean squared to compute error and keras' built-in `binary_accuracy` function for loss. This process found the optimal number of nodes to be `11`. I then trained the model again using the optimal number of nodes, however I set the epochs at `10,000` to allow for more learning. My final accuracy was approximately `0.65`.
#### load_data
The `load_data` function reads the data from the clean dataset and converts it into the necessary format for a keras neural network: a numpy array of type `float32`.
```
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
```
#### scale
The `scale` function transforms the features into the domain of `[0, 1]`, which normalizes the data and can improve model accuracy.
```
def scale(x_train):
	category_max = x_train[:,0].max()
	duration_max = x_train[:,1].max()
	goal_max = x_train[:,2].max()
	x_train[:,0] = [n / category_max for n in x_train[:,0]]
	x_train[:,1] = [n / duration_max for n in x_train[:,1]]
	x_train[:,2] = [n / goal_max for n in x_train[:,2]]
	return x_train
```
#### model_train
The `model_train` function builds and trains the keras model. For this API, the model is a sinlge layer neural network, using the sigmoid function. It accepts the tuning parameters of batch size, epoch size, learning rate, and number of nodes. Once the model has finished training it is exported to a specified file.
```
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
```
#### model_validate
The `model_validate` function attempts to fine the optimal number of nodes for a given batch sice, epoch size, and learn rate.The function accepts a minimum number of nodes, a maximum number of nodes, and a step, then trains the model across the set of nodes.
```
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
```
## Predict
To predict the success of a Kickstarter project, my project scrapes data from the the Kickstarter website and runs it through the trained model.
#### get_page
The `get_page` function returns the Kickstarter project data for the project most closely matching the search terms passed as a parameter. Since various Kickstarter pages have have differnt HTML layouts depending on how the browser and project author chose to format them I had difficulty creating a general scraper to pull data from the project page directly. However, the search projects page on Kickstarter is formatted consistently across all searches, so the `get_page` function pulls the raw JSON data returned as the result of a search. This has the added benifit of the a user only needing to know the relevant search terms to find their project, as opposed to a complete URL.
```
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
```
