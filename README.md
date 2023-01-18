# Team Process Mapping Take-Home Task: Yashveer Singh Sohi.

Goal: In this pre-test, you will implement a feature extractor that detects sentiment from team conversations. Specifically, given an input of conversation data, you should output: (1) a sentiment label of ‘positive,’ ‘negative,’ or ‘neutral,’ alongside a score for each label, from 0-1. You will run your feature extractor on a dataset of team jury conversations.

You will then write a reflection on how well you think this feature extractor performed on the data. Please write your reflection in this README document.

## 1. What method(s) did you choose?
In 1-2 sentences each, describe your sentiment analysis method(s).

> I stacked (arithmetic mean) scores from 2 different sentiment analysis models in order to generate a slightly robust sentiment classifier. We first use the following pretrained models from Huggingface to generate `positive`, `negative`, and `neutral` scores for the input text:
> - [ROBERTA Model trained on Tweets](https://huggingface.co/cardiffnlp/twitter-roberta-base-sentiment-latest)
>   - This is a general purpose sentiment classification model trained on a large courpus of tweets (~124M tweets).
> - [BERT Model trained on SemEval17 Corpus](https://huggingface.co/finiteautomata/bertweet-base-sentiment-analysis)
>   - This model is trained on a relatively smaller corpus of tweets (~40M) and other 3rd party data sources as compared to the first model, but was made specifically for NLP tasks in social conversations (for eg: chats)

> Once these model scores are available, we take an arithmetic mean of the probabilities (eg: `final_positive_prob` = average of `positive_prob_from_model_1` and `positive_prob_from_model_2`) while stacking the models together.

## 2. Method evaluation
Next, we would like you to consider how you would evaluate your method. How do you know the classification or quantification of emotion is “right?” Try to think critically!

2a. Open up output/jury_output_chat_level.csv and look at the columns you generated. Do the values “make sense” intuitively? Why or why not?

> On reviewing the stacked models scores, we can observe the following:
> - Strengths:
>   - The model generally classifies negative sentences correctly and with high confidence, and does classify positive and neutral statements correctly with slightly lesser confidence than the negative ones.
> - Areas of Improvements:
>   - Challenges around the word "a**hole": It can be used in a negative context while describing the subject of the reddit post. However, in neutral tones it is gets used to ask questions like `do you guys think that he is an a**hole?`. Since both the training data for both the models used in the final model are trained in general conversation where this word is typically used negatively, it skews our model scores distributions in favour of the negative sentiment more.
>   - Model is able to encorporate local context only: There are 2 ways in which this hurts model performance:
>     - Since these predictions are made on a chat level, we lose contextual information in the preceding messages of the participants. Sometimes participants think that the other characters in the story are in wrong more than the subject of the post itself, hence they use negative words for the other characters, which in this problem specifically should be treated as positive (for the subject). For eg: A sentence like, `I think that the mother in law is the a**hole here` would be classified as a negative comment, when infact this was made in favour of the subject of the post itself.
>     - The other place where long range context is needed by our models, is where we try and classificy sentences like `not an a**hole`, `not responsible for this..`, or `not in the wrong..`, etc. These sentences use 2 negatives to create an overall positive sentence, but this is somtimes lost by the model in long sentences, or when such conversations span over multiple chats.

> Conclusions:
> - It seems like, the model is able to classify sentiment reasonable well for some emotions. 
> - Due to an inherent bias in the model due to negative words, it would make sense to either correct for this factor post prediction, or try and create features pre-prediction that give more context to the model about when to wary of certain negative words being used in positive/neutral context.


2b. Propose an evaluation mechanism for your method(s). What metric would you use (e.g., F1, AUC, Accuracy, Precision, Recall)?

> I would evaluate this model in 2 stages as follows:
> - Stage 1: Are we predicting the correct sentiment?
>   - Being a multiclass classification problem, I would prefer using F1 score here. 
>   - As mentioned in the above sections, we have established that the model will perform better for negative snetiment sentences due to an inherent bias towards negative words that will invariably be used by the participants in this context. Hence, given that the performance on each class is expected to be different, I would consider "not" aggregating the F1 scores for each class into a single metric for the overall model. Doing this would give a false sense of performance especially for the positive and neutral classes. 
>   - In addition to this, the F1 score (calculated for each class separately) encorporates information from Precision, and Recall, and would compell the model to improve on both fronts, which is the goal of the model.
> - Stage 2: Are we predicting the correct sentiment with high confidence?:
>   - Once we are confident with the labels predictions by the model, I plan to optimize the predicted class probabilities using a `logloss` metric.
>   - Being able to confidently distinguish between the 3 classes is a skill that would pay large dividents by making the model robust when used to predict the sentiment of newer unseen chats.

2c. Describe the steps you would take in evaluating this method. Be as specific as possible.

> Broadly, the steps are as follows:
> - Before, we evaluate our model, we need to define labels that can be used as ground truth. Hence, initially I would work on annotating the messages as positive, negative or neutral.
> - Having labels would allow us to quantify and treat the bias towards negative sentiment in these chats. We can experiment with different preprocessing techniques such as replacing all "a**hole" words with a placeholder to see if the statements which used this word in a non-negative sentiment are being better classified or not. This would also harm the accuracy for the negative sentiment statements as we are masking some important context. Having labels allows us to quantify the tradeoff and make data driven decisions on how to address the inherent bias.
> - Once, we finalize the labels, and any preprocessing steps, we move towards model predictions (only class labels and not class probabilities for now), and calculate the F1 score for each class. 
> - Iteratively, we can analyse the errors and implement additional features and/or preprocessing to bump up the relevant F1 scores.
> - Once the F1 scores are at a reasonable stage, we can work on pushing the model probability scores to the extremes.
> - This can be achieved by penalizing "on-the-fence" probabilities by tweaking the logloss metric formulation while finetuning the model.
> - We can iteratively add in more features/heuristics to improve the probability scores as well.

2d. (OPTIONAL) Implement your proposed steps in 2b, using the DynaSent benchmark dataset (https://github.com/cgpotts/dynasent). Choose the Round 2 test dataset (“Sentences crowdsourced using Dynabench”). Evaluate the model using benchmarks you have proposed, and briefly comment on the quality of your performance.

> [YOUR ANSWER]

## 3. Overall reflection
3a. How much time did it take you to complete this task? (Please be honest; we are looking for feedback to make sure the task is scoped appropriately, as this is one of the first times we’re using this task.)

> About to 8-10 hours of work scattered across multiple sittings.

3b. Finally, provide an overall reflection of your experience. How did you approach this task? What challenge(s) did you encounter? If you had more time, what are additional extensions, improvements, or tests that you would want to implement?

> Overall, I really liked the structure of the test, and the thought and preperation went into making it a rewarding learning experience as well as a challenge! 

> The biggest blocker on the test for me was the model prediction time. The final model took about 1.5 hours to predict scores for 1000 chat messages. Hence, I had to execute the prediction functions on chunks of data in parallel to finish on time. Given that the dataset is intended to get bigger in future, and the team ideally would also be finetuning models in addition to just using them for inference, I imagine that this problem would only get worse, and I would suggest spending some time on developing easy to parallelize preciction functions to make iterations faster in the future.

> With additional time, I would look into assigning labels, as mentioned in 2.c, and use them to iteratively build preprocessing methods that could soften the bias on negative sentiment that I currently have in the model.
