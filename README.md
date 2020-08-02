<img src="https://i.imgur.com/uJyXJIs.png">

### Sorting Images by Likes with Log Decay Curve

```python
from numpy import exp
from time import time

SEC_IN_DAY = 60*60*24
```

 The <b>COEFFS </b> array contains the coefficients for three logarithmic decay curves.



```python
COEFFS = [
{
    'a':2,
    'b':11,
    'c':1,
},
{
    'a':1,
    'b':4,
    'c':1,
},
{
    'a':1/2,
    'b':1/2.11,
    'c':1.0308,
}]
```

The logarithmic decay curve is in the form of:
   
   <b>score = -1/20 * e^(a*dt - b) + c </b>
    
Where 
- <b>e</b> is the Euler's constant
- <b>dt</b> is the time difference between the timestamp of the score and the current timestamp
- <b>a, b, c</b> are the coefficients that specify the rate of decay


```python
def hot_score(timestamp, coeff):
    dt = time() - timestamp
    score =  -1/20 * exp(coeff['a'] * dt/SEC_IN_DAY - coeff['b']) + coeff['c']
    
    return (max(score, 0.0))
```

The three curves coefficients (<b>a, b,</b> and <b>c</b>) were determined with the criteria that a vote should start with a <b>score</b> of 1 if the vote was just cast, and end with a 0 when the vote is 7 days or older.


```python
for coeff in COEFFS:
    print ('score of vote just cast :       {}'.format(round(hot_score(time(), coeff),2)))
    print ('score of vote cast 7 days ago : {}'.format(round(hot_score(time()-7*SEC_IN_DAY, coeff),2)))
    print ('-')
```

    score of vote just cast :       1.0
    score of vote cast 7 days ago : 0.0
    -
    score of vote just cast :       1.0
    score of vote cast 7 days ago : 0.0
    -
    score of vote just cast :       1.0
    score of vote cast 7 days ago : 0.0
    -



```python
def total_hot_score(vote_timestamps, coeff):
    total_score = 0
    for timestamp in vote_timestamps:
        total_score += hot_score(timestamp, coeff)
        
    return total_score
```

The <b>total hot score</b> of a post is calculated by calculating the hot score of each one of it's votes and summing the resulting values.


In the example below, the post has 3 votes total cast 
- one vote cast 3 days ago
- one vote cast 4 days ago
- one vote cast 5 days ago


```python
vote_timestamps = [
    time()-3*SEC_IN_DAY,
    time()-4*SEC_IN_DAY,
    time()-5*SEC_IN_DAY
] 

for coeff in COEFFS:
    total_score = total_hot_score(vote_timestamps, coeff)
    print ('total score : {}'.format(round(total_score,2)))
    print ('-')
```

    total score : 2.98
    -
    total score : 2.8
    -
    total score : 2.34
    -


The example below also includes two additional votes, that are older than 7 days, and therefore do not impact the <b>total hot score</b> of the post.


```python
vote_timestamps = [
    time()-3*SEC_IN_DAY,
    time()-4*SEC_IN_DAY,
    time()-5*SEC_IN_DAY,
    time()-8*SEC_IN_DAY,
    time()-9*SEC_IN_DAY,
] 

for coeff in COEFFS:
    total_score = total_hot_score(vote_timestamps, coeff)
    print ('total score : {}'.format(round(total_score,2)))
    print ('-')
```

    total score : 2.98
    -
    total score : 2.8
    -
    total score : 2.34
    -


The example below shows the <b>total hot score</b> if the two additional votes were placed within the past 7 days.


```python
vote_timestamps = [
    time()-3*SEC_IN_DAY,
    time()-4*SEC_IN_DAY,
    time()-5*SEC_IN_DAY,
    time()-6*SEC_IN_DAY,
    time()-6.5*SEC_IN_DAY,
] 

for coeff in COEFFS:
    total_score = total_hot_score(vote_timestamps, coeff)
    print ('total score : {}'.format(round(total_score,2)))
    print ('-')
```

    total score : 4.47
    -
    total score : 3.82
    -
    total score : 2.98
    -
