# PosterRecognition

yolo 1 + yolo 2

Using deep neural network ([Darknet YOLO](http://pjreddie.com/darknet/)) for academic poster recognition. This repo is for a Lafayette EXCEL research with Dr. Amir Sadovnik.



INSTRUCTION

----------------------
On the server side, the tree view for some important folders inside the project root /home/vut/PosterRecognition/ is as follows.

PosterRecognition (project-root)
|-DeepNet
|---darknet
|-----PosterRecognition			# github repo containing src code
|-------darknet-yolo1			# yolo1 is no longer in use, but the scripts for generating training data is in here
|---------scripts				# scripts for generating training data and visualizing the final result
|-----------training
|-------------temp
|-------darknet-yolo2 			# src code for training/ testing the deep neural network on poster recognition
|---------cfg
|-----------yolov1
|---------src
|-------feature-matching 		# scripts to run opencv feature matching (to compare results with that of the DNN)
|---Database 					# database: input and output data
|-----featureMatching 			# data for opencv feature matching
|-------results
|-------srcPosters
|-------test
|-------train
|-----realworld					# data for deep neural network
|-------set2
|---------docs					# documentation, for presentation purpose
|-----------blur
|-----------light
|-----------occ
|-----------perspective
|-----------rotation
|-----------scaleNtranslate
|-----------training
|-------------temp
|---------randTest				# input data for testing
|-----------100C_2000P_trial3 	# example experiment
|-------------JPEGImages
|---------randTrain				# input and output data of training
|-----------100C_2000P_trial3	# example experiment
|-------------backup
|---------------classify_weights
|---------------detect_weights
|---------------yolo2_weights
|-------------JPEGImages
|-------------labels
|---------src 					# original source posters
|-trash

----------------------
Generate training and testing data:
1. Move to darknet-yolo1 folder: cd <project-root>/DeepNet/darknet/PosterRecognition/darknet-yolo1/scripts/training/

2. Provide input parameters in config.py. ex:
	CLASSES = 100
	NUM_VAR = 2000
	TRIAL = 3

3. Run: python main.py

4. Output data should be in 
		<project-root>/DeepNet/database/realworld/set2/randTrain/ 
   and 
		<project-root>/DeepNet/database/realworld/set2/randTest/

----------------------
Train: 
1. Move to darknet-yolo2 folder: cd <project-root>/DeepNet/darknet/PosterRecognition/darknet-yolo2/

2. Prepare config file
	a. Copy and rename the sample config file ./cfg/yolo2_80c.cfg to 
			<project-root>/DeepNet/database/realworld/set2/randTrain/cfg-yolo2/yolo2_Xc.cfg
	   with X = number of classes (posters)
	b. Adjust "classes=X" in the [region] layer (at the end of the file)
	c. Adjust "filters=Y" in the last [convolutional] layer, with Y = (X + 5) * 5
	d. Adjust any other desired parameters, such as max_batches, learning_rate, ...
	e. ex: <project-root>/DeepNet/database/realworld/set2/randTrain/cfg-yolo2/yolo2_100c.cfg
		[net]
		...
		max_batches = 120000
		...

		[convolutional]
		...
		filters=525
		...

		[region]
		...
		classes=100
		...

3. Adjust params in train_detector() method in .src/detector.c
	a. cfg_classes, cfg_imgs_per_class, note should match the parameters used to generate training/ testing data
	b. Ex: 	cfg_classes = 100
			cfg_imgs_per_class = 2000
			note = "trial3"

2. Run: make

3. Run: ./darknet detector train x x x -gpus 0 1 2 3

4. The output weights should be in <project-root>/DeepNet/database/realworld/set2/randTrain/100C_2000P_trial3/backup/yolo2_weights/
	yolo2_100c_100.weights
	yolo2_100c_200.weights
	...
	yolo2_100c_1000.weights
	yolo2_100c_2000.weights
	...
	yolo2_100c_<max_batches>.weights
	yolo2_100c_final.weights

----------------------
Validation 
1. Adjust parameters in multivalidate_detector() method in ./src/detector.c, ex:
	a. The params should match those used for training
	c. To save images as visualization while validating, set savingImg = 1 (see Visualization section below). If validating many weight files, set savingImg to 0 to save runtime.
	b. Ex: 	cfg_classes = 100
			note = "trial3"
			cfg_imgs_per_class = 2000
			int start_weight = 1
			int end_weight = <max_batches>/1000
			savingImg = 0 

2. Run: make

3. Run: ./darknet detector multivalid x x x 

4. The output results should be in <project-root>/DeepNet/database/realworld/set2/randTest/100C_2000P_trial3/testYolo2_<time-stamp>.txt

----------------------
Visualization
1. To visualize the ouput of some weight files, do validation with "savingImg = 1"
	Ex: 	cfg_classes = 100
			note = "trial3"
			cfg_imgs_per_class = 2000
			int start_weight = 100
			int end_weight = 100	// visualize results of yolo2_100c_100000.weights only
			savingImg = 0 

2. The output images should be in <project-root>/DeepNet/database/realworld/set2/randTest/100C_2000P_trial3/results

3. Copy the following file and folders into the same directory:
	a. <project-root>/DeepNet/darknet/PosterRecognition/darknet-yolo1/scripts/displayResults.py
	b. <project-root>/DeepNet/database/realworld/set2/randTest/100C_2000P_trial3/results
	c. <project-root>/DeepNet/database/realworld/set2/src

4. Run: python displayResults.py

5. The output visualization is results.html

