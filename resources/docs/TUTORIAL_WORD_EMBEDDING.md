# Tutorial 3: Word Embeddings

We provide a set of classes with which you can embed the words in sentences in various ways. This tutorial explains
how that works. We assume that you're familiar with the [base types](/resources/docs/TUTORIAL_BASICS.md) of this library.  


# Embeddings

All embedding 
classes inherit from the `TextEmbeddings` class and implement the `embed()` method which you need to call 
to embed your text. This means that for most users of Flair, the complexity of different embeddings remains hidden 
behind this interface. Simply instantiate the embedding class you require and call `embed()` to embed your text.

All embeddings produced with our methods are pytorch vectors, so they can be immediately used for training and 
fine-tuning.

## Classic Word Embeddings

Classic word embeddings are static and word-level, meaning that each distinc word gets exactly one pre-computed 
embedding. Most embeddings fall under this class, including the popular GloVe or Komnios embeddings. 

Simply instantiate the WordEmbeddings class and pass a string identifier of the embedding you wish to load. So, if 
you want to use GloVe embeddings, pass the string 'glove' to the constructor: 

```python
# all embeddings inherit from the TextEmbeddings class. Init a simple glove embedding.
from flair.embeddings import WordEmbeddings
glove_embedding = WordEmbeddings('glove')
```
Now, create an example sentence and call the embedding's `embed()` method. You always pass a list of sentences to 
this method since some embedding types make use of batching to increase speed. So if you only have one sentence, 
pass a list containing only one sentence:

```python
# embed a sentence using glove.
from flair.data import Sentence
sentence = Sentence('The grass is green .')
glove_embedding.embed(sentences=[sentence])

# now check out the embedded tokens.
for token in sentence:
    print(token)
    print(token.embedding)
```

This prints out the tokens and their embeddings. GloVe embeddings are pytorch vectors of dimensionality 100.

You choose which pre-trained embeddings you load by passing the appropriate 
string you pass to the constructor of the `WordEmbeddings` class. Currently, the following static embeddings
are provided (more coming): 
 
| ID | Language | Embedding | 
| ------------- | -------------  | ------------- |
| 'en-glove' (or 'glove') | English | GloVe embeddings |
| 'en-numberbatch' (or 'numberbatch') | English |[Numberbatch](https://github.com/commonsense/conceptnet-numberbatch) embeddings |
| 'en-extvec' (or 'extvec') | English |Komnios embeddings |
| 'en-crawl' (or 'crawl')  | English |FastText embeddings over Web crawls |
| 'en-news' (or 'news')  |English | FastText embeddings over news and wikipedia data |
| 'de-fasttext' | German |German FastText embeddings |
| 'de-numberbatch' |German | German Numberbatch embeddings |
| 'sv-fasttext' |Swedish | Swedish FastText embeddings |

So, if you want to load German FastText embeddings, instantiate the method as follows:

```python
german_embedding = WordEmbeddings('de-fasttext')
```

## Contextual String Embeddings


Contextual string embeddings are [powerful embeddings](https://drive.google.com/file/d/17yVpFA7MmXaQFTe-HDpZuqw9fJlmzg56/view?usp=sharing)
 that capture latent syntactic-semantic information that goes beyond
standard word embeddings. Key differences are: (1) they are trained without any explicit notion of words and
thus fundamentally model words as sequences of characters. And (2) they are **contextualized** by their
surrounding text, meaning that the *same word will have different embeddings depending on its
contextual use*.

With Flair, you can use these embeddings simply by instantiating the appropriate embedding class, same as before:

```python

# the CharLMEmbedding also inherits from the TextEmbeddings class
from flair.embeddings import CharLMEmbeddings
charlm_embedding_forward = CharLMEmbeddings('news-forward')

# embed a sentence using CharLM.
from flair.data import Sentence
sentence = Sentence('The grass is green .')
charlm_embedding_forward.embed(sentence)
```

You choose which embeddings you load by passing the appropriate string to the constructor of the `CharLMEmbeddings` class. 
Currently, the following contextual string embeddings are provided (more coming):
 
| ID | Language | Embedding | 
| -------------     | ------------- | ------------- |
| 'news-forward'    | English | Forward LM embeddings over 1 billion word corpus |
| 'news-backward'   | English | Backward LM embeddings over 1 billion word corpus |
| 'mix-forward'     | English | Forward LM embeddings over mixed corpus (Web, Wikipedia, Subtitles) |
| 'mix-backward'    | English | Backward LM embeddings over mixed corpus (Web, Wikipedia, Subtitles) |
| 'german-forward'  | German  | Forward LM embeddings over mixed corpus (Web, Wikipedia, Subtitles) |
| 'german-backward' | German  | Backward LM embeddings over mixed corpus (Web, Wikipedia, Subtitles) |

So, if you want to load embeddings from the English news backward LM model, instantiate the method as follows:

```python
charlm_embedding_backward = CharLMEmbeddings('news-backward')
```


## Character Embeddings

Some embeddings - such as character-features - are not pre-trained but rather trained on the downstream task. Normally
this requires you to implement a [hierarchical embedding architecture](http://neuroner.com/NeuroNERengine_with_caption_no_figure.png). 

With Flair, you need not worry about such things. Just choose the appropriate
embedding class and character features will then automatically train during downstream task training. 

```python
# the CharLMEmbedding also inherits from the TextEmbeddings class
from flair.embeddings import CharacterEmbeddings
embedder = CharacterEmbeddings()

# embed a sentence using CharLM.
from flair.data import Sentence
sentence = Sentence('The grass is green .')
embedder.embed(sentence)
```

# Stacked Embeddings

Stacked embeddings are one of the most important concepts of this library. You can use them to combine different embeddings
together, for instance if you want to use both traditional embeddings together with contextual sting embeddings. 
Stacked embeddings allow you to mix and match. We find that a combination of embeddings often gives best results. 

All you need to do is use the `StackedEmbeddings` class and instantiate it by passing a list of embeddings that you wish 
to combine. For instance, lets combine classic GloVe embeddings with embeddings from a forward and backward 
character language model.

First, instantiate the three embeddings you wish to combine: 

```python
# the CharLMEmbedding also inherits from the TextEmbeddings class
from flair.embeddings import WordEmbeddings, CharLMEmbeddings

# init GloVe embedding
glove_embedding = WordEmbeddings('glove')

# init CharLM embedding
charlm_embedding_forward = CharLMEmbeddings('news-forward')
charlm_embedding_backward = CharLMEmbeddings('news-backward')
```

Now instantiate the `StackedEmbeddings` class and pass it a list containing these three embeddings.

```python
# now create the StackedEmbedding object that combines all embeddings
from flair.embeddings import StackedEmbeddings
stacked_embeddings = StackedEmbeddings(embeddings=[glove_embedding, charlm_embedding_forward, charlm_embedding_backward])
```

That's it! Now just use this embedding like all the other embeddings, i.e. call the `embed()` method over your sentences.

```python
# just embed a sentence using the StackedEmbedding as you would with any single embedding.
from flair.data import Sentence
sentence = Sentence('The grass is green .')
stacked_embeddings.embed(sentence)

# now check out the embedded tokens.
for token in sentence:
    print(token)
    print(token.embedding)
```

Words are now embedding using a concatenation of three different embeddings. This means that the resulting embedding
vector is still a single Pytorch vector. 


## Next 

You can now either look into [text embeddings](/resources/docs/TUTORIAL_TEXT_EMBEDDINGS.md) to embed entire text passages
with one vector for tasks such as text classification, or go directly to the tutorial about 
[training your own models](/resources/docs/TUTORIAL_TRAINING_A_MODEL.md). 

