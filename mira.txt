# mira.py
# -------
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


# Mira implementation
import util, random
PRINT = True

class MiraClassifier:
    """
    Mira classifier.

    Note that the variable 'datum' in this code refers to a counter of features
    (not to a raw samples.Datum).
    """
    def __init__( self, legalLabels, max_iterations):
        self.legalLabels = legalLabels
        self.type = "mira"
        self.automaticTuning = False
        self.C = 0.001
        self.legalLabels = legalLabels
        self.max_iterations = max_iterations
        self.initializeWeightsToZero()

    def initializeWeightsToZero(self):
        "Resets the weights of each label to zero vectors"
        self.weights = {}
        for label in self.legalLabels:
            self.weights[label] = util.Counter() # this is the data-structure you should use

    def train(self, trainingData, trainingLabels, validationData, validationLabels):
        "Outside shell to call your method. Do not modify this method."

        self.features = trainingData[0].keys() # this could be useful for your code later...

        if (self.automaticTuning):
            Cgrid = [0.002, 0.004, 0.008]
        else:
            Cgrid = [self.C]

        return self.trainAndTune(trainingData, trainingLabels, validationData, validationLabels, Cgrid)
    
    """
    # Author    : Ashish Kumar
    # Version   : v1.0
    # Date      : 2-Dec-2015
    """
    def trainAndTune(self, trainingData, trainingLabels, validationData, validationLabels, Cgrid):
        """
        This method sets self.weights using MIRA.  Train the classifier for each value of C in Cgrid,
        then store the weights that give the best accuracy on the validationData.

        Use the provided self.weights[label] data structure so that
        the classify method works correctly. Also, recall that a
        datum is a counter from features to values for those features
        representing a vector of values.
        """
        "*** YOUR CODE HERE ***"
        
        bestC = -0.01 # Given C is a positive constant so, we can initialize with a negative value
        bestAccuracy = -0.01 
        bestWeights = {}
        
        # iterate over different values of
        for c in Cgrid:
            for iteration in range(self.max_iterations):
                print "Starting iteration ", iteration, ", c =", c , "..."
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
                        tempFeatures = features.copy()
                        numerator = ((self.weights[labelWithHighestScore] - self.weights[trueLabel]) * features ) + 1.0
                        denominator = (2.0 * (features * features))
                        
                        try: 
                            t = numerator/denominator
                        except:
                            t = 0
                            
                        # tau calculation
                        t = min(c, t)
                        
                        for key in tempFeatures:
                            tempFeatures[key] *= t
                        
                        self.weights[trueLabel] = self.weights[trueLabel] + tempFeatures
                        self.weights[labelWithHighestScore] = self.weights[labelWithHighestScore] - tempFeatures
            
            numberOfCorrectClassifications = 0
            guesses = self.classify(validationData)
            
            # calculate the total number of correct classification
            for i in range(len(guesses)):
                if validationLabels[i] == guesses[i]:
                    numberOfCorrectClassifications = numberOfCorrectClassifications + 1
            
            # calculate the accuracy
            accuracy = (numberOfCorrectClassifications * 100)/len(guesses)
            print "Accuracy for c =", c , "is", accuracy , "%" 
            if accuracy > bestAccuracy:
                bestAccuracy = accuracy
                bestC = c
                bestWeights = self.weights
            elif accuracy == bestAccuracy:
                if bestC > c:
                    bestAccuracy = accuracy
                    bestC = c
                    bestWeights = self.weights
                    
                                
        self.C = bestC
        self.weights = bestWeights    
                        
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


