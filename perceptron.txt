# perceptron.py
# -------------
# Licensing Information:  You are free to use or extend these projects for
# educational purposes provided that (1) you do not distribute or publish
# solutions, (2) you retain this notice, and (3) you provide clear
# attribution to UC Berkeley, including a link to http://ai.berkeley.edu.
# 
# Attribution Information: The Pacman AI projects were developed at UC Berkeley.
# The core projects and autograders were primarily created by John DeNero
# (denero@cs.berkeley.edu) and Dan Klein (klein@cs.berkeley.edu).
# Student side autograding was added by Brad Miller, Nick Hay, and
# Pieter Abbeel (pabbeel@cs.berkeley.edu).


# Perceptron implementation
import util, random
PRINT = True

class PerceptronClassifier:
    """
    Perceptron classifier.

    Note that the variable 'datum' in this code refers to a counter of features
    (not to a raw samples.Datum).
    """
    def __init__( self, legalLabels, max_iterations):
        self.legalLabels = legalLabels
        self.type = "perceptron"
        self.max_iterations = max_iterations
        self.weights = {}
        for label in legalLabels:
            self.weights[label] = util.Counter() # this is the data-structure you should use

    def setWeights(self, weights):
        assert len(weights) == len(self.legalLabels);
        self.weights = weights;

    """
    # Author    : Ashish Kumar
    # Version   : v1.0
    # Date      : 2-Dec-2015
    """
    def train( self, trainingData, trainingLabels, validationData, validationLabels ):
        """
        The training loop for the perceptron passes through the training data several
        times and updates the weight vector for each label based on classification errors.
        See the project description for details.

        Use the provided self.weights[label] data structure so that
        the classify method works correctly. Also, recall that a
        datum is a counter from features to values for those features
        (and thus represents a vector a values).
        """

        self.features = trainingData[0].keys() # could be useful later
        # DO NOT ZERO OUT YOUR WEIGHTS BEFORE STARTING TRAINING, OR
        # THE AUTOGRADER WILL LIKELY DEDUCT POINTS.

        for iteration in range(self.max_iterations):
            print "Starting iteration ", iteration, "..."
            for i in range(len(trainingData)):
                "*** YOUR CODE HERE ***"
                features = trainingData[i]
                
                # these will be updated in each iteration
                # will contain the highest score and the label with 
                # the highest score
                labelWithHighestScore = -1
                highestScore = -1
                
                # randomize the labels
                tempLabels = list(self.legalLabels)
                tempLabels = sorted(tempLabels, key=lambda k: random.random())
                
                # iterate over the randomized labels
                for label in tempLabels:
                    score = features * self.weights[label]
                    
                    # select the highest score
                    if highestScore == -1:
                        highestScore = score
                        labelWithHighestScore = label
                    elif score > highestScore:
                        highestScore = score
                        labelWithHighestScore = label
                
                # fetch the true label
                trueLabel = trainingLabels[i]
                
                # check the calculated label against the true label
                if labelWithHighestScore != trueLabel:
                    self.weights[trueLabel] = self.weights[trueLabel] + features
                    self.weights[labelWithHighestScore] = self.weights[labelWithHighestScore] - features
                        
                

    def classify(self, data ):
        """
        Classifies each datum as the label that most closely matches the prototype vector
        for that label.  See the project description for details.

        Recall that a datum is a util.counter...
        """
        guesses = []
        for datum in data:
            vectors = util.Counter()
            for l in self.legalLabels:
                vectors[l] = self.weights[l] * datum
            guesses.append(vectors.argMax())
        return guesses

    """
    # Author    : Ashish Kumar
    # Version   : v1.0
    # Date      : 2-Dec-2015
    """
    def findHighWeightFeatures(self, label):
        """
        Returns a list of the 100 features with the greatest weight for some label
        """
        featuresWeights = []

        "*** YOUR CODE HERE ***"
        #select the weight corresponding to the given label
        weight = self.weights[label];
        #use function in the counter class to fetch the sorted keys
        sortedKeys = weight.sortedKeys();
        #push the highest valued keys in the required array
        for i in range(0, 99):
            featuresWeights.append(sortedKeys[i])

        return featuresWeights
