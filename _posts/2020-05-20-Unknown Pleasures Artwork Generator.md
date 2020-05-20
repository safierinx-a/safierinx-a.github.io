# Unknown Pleasures Artwork Style Converter
Joy Division's Unknown Pleasures Album Art is perhaps one of the most iconnic album artworks out there. The 40th anniversary of Ian Curtis' death was a couple of days ago so I decided to post this little script I'd written some time ago as a quick little exercise.


```python
path = r"C:\Users\APOORV\Downloads\joy division.jpg"
picture = Image.open(path)
picture
```




![](/post_resources/JD/output_1_0.png)



This isn't particularly hard but I like it all the same. We'll be using the matplotlib, math, numpy and PIL libraries here. You know, bread and butter stuff.


```python
import matplotlib.image as img 
import matplotlib.pyplot as plt 
import numpy as np
import math
from PIL import Image
```

Let's use a generic butterfly image to play around with.


```python
path = r"C:\Users\APOORV\Downloads\GettyImages-182457909-f1e4c7a.jpg"
picture = Image.open(path)
rad = np.asarray(picture)
rad = rad[:,:,0]
rad = 255*rad
picture
```




![](/post_resources/JD/output_5_0.png)



Next, we'll construct an array taking pixel values from evenly spaced rows and dividing by 255 to convert the corresponding amplitude to be between 0 and 1. Play around with the value of t to change spacing of the rows.


```python
sad = []
t = int(pow(math.sqrt(np.shape(rad)[0]), .65))
c = len(rad);
for i in range(len(rad)):
    if i%t==0:
        sad.append(rad[i]/255+c)
        c-=1
r = np.arange(0, len(sad[0]))
ax = plt.axes()
# Setting the background color
ax.set_facecolor("black")

for i in sad:
    plt.plot(r, i, 'white')
```


![](/post_resources/JD/output_7_0.png)


That's basically it. I've been working on a few other artwork generators with a friend of mine, perhaps I'll put up more if I feel like it. I'm on the look out for more of such artwork I could replicate. Let me know if you know of any!
