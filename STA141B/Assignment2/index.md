
# Assignment 2

## Part 1: Image Processing Basics

Computers use tiny dots called _pixels_ to display images. Each pixel is stored as an array of numbers that represent color intensities.

__Example.__ In an 8-bit grayscale image, each pixel is a single number. The number represents light intensity ranging from black (0) to white (255).

__Example.__ In a 24-bit RGB color image, each pixel is an array of 3 numbers. These numbers range from 0 to 255 and represent red, green, and blue intensity, respectively. For instance, `(0, 0, 255)` is <span style="color:#00F">bright blue</span> and `(255, 128, 0)` is <span style="color:#FF8000">orange</span>.

In this assignment, you'll use Python and NumPy to manipulate 24-bit RGB color images.

You can use `Image.open()` from the Python imaging library (PIL) to open an image:


```python
from PIL import Image

# Cat image from https://unsplash.com/photos/FqkBXo2Nkq0
cat_img = Image.open("cat.png")
```

Images display inline in Jupyter notebooks:


```python
cat_img
```




![png](output_4_0.png)



In a Python terminal, you can display the image in a new window with `.show()` instead.

NumPy can convert images to arrays:


```python
import numpy as np

cat = np.array(cat_img)
```

To convert an array back to an image (for display) use the function below:


```python
def as_image(x):
    """Convert an ndarray to an Image.
    
    Args:
        x (ndarray): The array of pixels.
        
    Returns:
        Image: The Image object.
    """
    return Image.fromarray(np.uint8(x))
```

__Exercise 1.1.__ How many dimensions does the `cat` array have? What does each dimension represent?


```python
#print cat
print cat.ndim
```

    3


__Answer:__ The cat array has 3 dimensions. The first represents red, the send represents green, and the third represents blue. 

__Exercise 1.2.__ Use `.copy()` to copy the cat array to a new variable. Swap the green and blue color channels in the copy. Display the result.


```python
copyCat = cat.copy()
#(267,400,3)

for x in range(copyCat.shape[0]): #get x positions
    for y in range(copyCat.shape[1]): #get y positions
        pixelArray = copyCat[x][y] #get values at position x,y
        copyCat[x][y] = [pixelArray[0], pixelArray[2], pixelArray[1]] #swap green and blue color channels in copy

copyCat_img = as_image(copyCat)
```


```python
copyCat_img
```




![png](output_14_0.png)



__Exercise 1.3.__ Why is `.copy()` necessary in exercise 1.2? What happens if you don't use `.copy()`?

__Answer:__ .copy() is necessary because if we did not, we would simply be creating another object that cat points to. By making a copy of cat, we make it a completely different object in memory.

__Exercise 1.4.__ Flip the blue color channel from left to right. Display the resulting image. _Hint: see the NumPy documentation on array manipulation routines._


```python
copyCat2 = cat.copy()
flipCat = np.fliplr(copyCat2) 

for x in range(copyCat2.shape[0]): #get x position
    for y in range(copyCat2.shape[1]): #get y position
        copyCat2[x,y,2] = flipCat[x,y,2] #move blue channel from left to right using flipCat
        
        

#print flip
copyCat2_img = as_image(copyCat2)
copyCat2_img
```




![png](output_18_0.png)



## Part 2: Singular Value Decomposition

Suppose $X$ is an $n \times p$ matrix (for instance, one color channel of the cat image). The _singular value decomposition_ (SVD) factors $X$ as $X = UD V^T$, where:

* $U$ is an $n \times n$ orthogonal matrix
* $D$ is an $n \times p$ matrix with zeroes everywhere except the diagonal
* $V$ is an $p \times p$ orthogonal matrix

Note that a matrix $A$ is _orthogonal_ when $A^T A = I$ and $AA^T = I$.

__Example.__ We can use NumPy to compute the SVD for a matrix:


```python
x = np.array(
    [[0, 2, 3],
     [3, 2, 1]]
)
u, d, vt = np.linalg.svd(x)
# Here d is 2x2 because NumPy only returns the diagonal of D.
print "u is:\n", u, "\nd is:\n", d, "\nv^T is:\n", vt
```

    u is:
    [[-0.68145174 -0.73186305]
     [-0.73186305  0.68145174]] 
    d is:
    [ 4.52966162  2.54600974] 
    v^T is:
    [[-0.48471372 -0.62402665 -0.6128975 ]
     [ 0.80296442 -0.03960025 -0.59470998]
     [ 0.34684399 -0.78039897  0.52026598]]


If we let

* $u_i$ denote the $i$th column of $U$
* $d_i$ denote the $i$th diagonal element of $D$
* $v_i$ denote the $i$th column of $V$

then we can write the SVD as $\ X = UDV^T = d_1 u_1 v_1^T + \ldots + d_m u_m v_m^T\ $ using the rules of matrix multiplication. In other words, the SVD decomposes $X$ into a sum!

If we eliminate some of the terms in the sum, we get a simple approximation for $X$. For instance, we could eliminate all but first 3 terms to get the approximation $X \approx d_1 u_1 v_1^T + d_2 u_2 v_2^T + d_3 u_3 v_3^T$. This is the same as if we:

* Zero all but the first 3 diagonal elements of $D$ to get $D_3$, then compute $X \approx UD_3V^T$
* Eliminate all but the first 3 columns of $V$ to get $p \times 3$ matrix $V_3$, then compute $X \approx UDV_3^T$

We always eliminate terms starting from the end rather than the beginning, because these terms contribute the least to $X$.

Why would we want to approximate a matrix $X$?

In statistics, _principal components analysis_ uses this approximation to reduce the dimension (number of covariates) in a  centered (mean 0) data set. The vectors $d_i u_i$ are called the _principal components_ of $X$. The vectors $v_i^T$ are called the _basis vectors_. Note that both depend on $X$. The dimension is reduced by using the first $q$ principal components instead of the original $p$ covariates. In other words, the $n \times p$ data $X$ is replaced by the $n \times q$ data $UD_q = XV_q$

In computing, this approximation is sometimes used to reduce the number of bits needed to store a matrix (or image). If $q$ terms are kept, then only $nq + pq$ values (for $XV_q$ and $V_q^T$) need to be stored instead of the uncompressed $np$ values.

__Exercise 2.1.__ Write the functions described below.

* A function that takes a matrix $X$ and returns its principal component matrix $XV_q$ and basis matrix $V_q^T$. This function should also take the number of terms kept $q$ as an argument.

* A function that takes a principal component matrix $XV_q$ and basis matrix $V_q^T$ and returns an approximation $\hat{X}$ for the original matrix.

As usual, make sure to document your functions. Test your function on the red color channel of the cat image. What's the smallest number of terms where the cat is still recognizable as a cat?


```python
def singValDecomp(x,q):
    u, d, vt = np.linalg.svd(x)
    
    d = d[:q] # strip values past q
    d = np.diag(d) #turns it into the full matrix
#     print d
    
    d_approx = np.zeros((x.shape[0], x.shape[1])) #matrix of zeros that's the size of the original x matrix
    d_approx[:d.shape[0],:d.shape[1]] = d #fill upper left corner with d
    
    
    vtq = vt.copy()

    
    xvq = np.dot(x, vtq.T)
    xvq = np.dot(u, d_approx)
#     print "xvq\n", xvq
    
    return xvq , vtq

def approximation(xvq, vtq):
    return np.dot(xvq, vtq)
```


```python
xvq, vtq = singValDecomp(x,1)
approximation(xvq, vtq)
```




    array([[ 1.49618805,  1.92621165,  1.89185877],
           [ 1.60687057,  2.06870575,  2.03181157]])




```python
copyCat3 = cat.copy()

copyCat3[:,:,1] = np.zeros((copyCat3.shape[0], copyCat3.shape[1])) #taking out blue
copyCat3[:,:,2] = np.zeros((copyCat3.shape[0], copyCat3.shape[1])) #taking out green

red = copyCat3[:,:,0] #isolating red

def terms(x,q):
    xvq, vtq = singValDecomp(x, q)
    approx_x = approximation(xvq, vtq)
    return approx_x.astype(np.int8)

copyCat3[:,:,0] = terms(red,18) #18 terms of red

as_image(copyCat3)
```




![png](output_24_0.png)



The cat is still recognizable with the first 18 terms. 

 __Exercise 2.2.__ You can check the number of bytes used by a NumPy array with the `.nbytes` attribute. How many bytes does the red color channel of the cat image use? How many bytes does the compressed version use when 10 terms are kept? What percentage of the original size is this?


```python
print "Original red color channel :", red.nbytes
print "Bytes with 10 terms :", terms(red,10).nbytes
percentage = (terms(red,10).nbytes) / (red.nbytes)
print "Percentage of original size : ", percentage *100 ,"%"

```

    Original red color channel : 106800
    Bytes with 10 terms : 106800
    Percentage of original size :  100 %


We expected the percentage to be smaller, but we ended up keeping arrays in which there are more trailing 0's than we thought, thus causing the trimmed down array to be about the same size as the original. I could not figure out which component in my SVD calculatins contributed to this. This deeply disappoints me. 
