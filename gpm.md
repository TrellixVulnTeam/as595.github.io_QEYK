---
layout: page
permalink: /gpm/
---

I'm guessing that most people are pretty comfortable with the concept of uncorrelated Gaussian noise. It's the most frequently assumed noise. Even if you don't realise it, you're probably assuming Gaussian noise.

Quick check: Are you using a chi-squared test to fit your data? Yes? Well there you go.
<h3>Co-variate Gaussian Noise</h3>
Here I'm going to talk about <strong>multi-variate</strong>, or <strong>co-variate</strong>, Gaussian noise. Co-variate Gaussian noise is the situation where <em>the value of one data point affects the value of another</em>. This kind of <em>co-</em>variance, i.e. variance <em>between</em> things, is usually expressed as a <strong>covariance matrix</strong>.

If you have <strong>N</strong> data points, then your covariance matrix will have a size: <strong>N x N</strong>. The matrix is normally denoted <strong>K</strong> (or sometimes $latex \mathbf{\Sigma}$) .
<p style="text-align:center;">$latex \mathbf{K(x,x)} = \left( \begin{array}{cccc} k(x_1,x_1) & k(x_1,x_2) & ... & k(x_1,x_n) \\ k(x_2,x_1) & k(x_2,x_2) & ... & k(x_2,x_n) \\ \vdots & \vdots & \vdots & \vdots \\ k(x_n,x_1) & k(x_n,x_2) & ... & k(x_n,x_n)  \end{array} \right) $</p>
<strong>Uncorrelated</strong>, or <em>independent</em>, Gaussian noise is a special case of the covariance matrix where only the diagonal elements have a non-zero value, i.e.
<p style="text-align:center;">$latex \mathbf{K(x,x)} = \left( \begin{array}{cccc} k(x_1,x_1) & 0 & ... & 0 \\ 0 & k(x_2,x_2) & ... & 0 \\ \vdots & \vdots & \vdots & \vdots \\ 0 & 0 & ... & k(x_n,x_n)  \end{array} \right) $</p>
The value of each diagonal element corresponds to the variance of a particular data point, e.g. a data point at position $latex x_1$ with a mean value $latex \mu_1 $ would have a variance $latex k(x_1,x_1) = \sigma_1^2 $ ; a data point at position $latex x_2$ with a mean value $latex \mu_2 $ would have a variance $latex k(x_2,x_2) = \sigma_2^2 $ , and so on and so forth.

The actual <em>y</em>-value at each <em>x</em>-position will be drawn from a probability distribution
<p style="text-align:center;">$latex p(y_i) = N (\mu_i, K_{ij})$</p>
which in the case of a diagonal covariance matrix reduces to
<p style="text-align:center;">$latex y_i = \mu_i + N(0, \sigma^2_i)$.</p>
However, if we start to add in non-zero values to the other elements of the covariance matrix then this will no longer be the case and the <em>y</em>-value at one position will affect the <em>y</em>-value at another.

This is a very brief explanation of covariate Gaussian noise. For a better and more detailed description, I like these references:
<ul>
	<li><a href="http://www.gaussianprocess.org" target="_blank" rel="noopener">Gaussian Processes for Machine Learning</a>, Carl Edward Rasmussen and Chris Williams, the MIT Press</li>
	<li><a href="http://www.robots.ox.ac.uk/~sjrob/Pubs/philTransA_2012.pdf" target="_blank" rel="noopener">Gaussian processes for time-series modelling</a>, S. Roberts, M. Osborne, M. Ebden, S. Reece, N. Gibson and S. Aigrain, Phil. Trans. R. Soc. A 2013 371, 20110550</li>
</ul>
The second of these has a particularly nice figure showing the effect of covariate Gaussian noise. It looks like this:

[caption id="attachment_1902" align="aligncenter" width="469"]<img class="alignnone  wp-image-1902" src="https://allofyourbases.files.wordpress.com/2017/08/roberts_fig5.png" alt="roberts_fig5" width="469" height="176" /> Figure 5 from Roberts et al. 2012[/caption]

Here I'm going to explain how to recreate this figure using Python.
<h3>Covariate Gaussian Noise in Python</h3>
To simulate the effect of co-variate Gaussian noise in Python we can use the <a href="http://www.numpy.org/" target="_blank" rel="noopener">numpy</a> library function <code>multivariate_normal(mean,K)</code>.

<em>Note: the Normal distribution and the Gaussian distribution are the same thing.</em>

First off, let's load some libraries:

[code language="python" gutter="false" highlight="1-100"]
import numpy as np   # the numpy library
import pylab as pl   # the matplotlib for plotting
[/code]

then we can call the <code>multivariate_normal(mean,K)</code> function:

[code language="python" gutter="false" highlight="1-100"]
# make an array of positions
# these are evenly spaced, but they don’t have to be
x = np.arange(0, 20.,0.01)

# use numpy to draw a sample from the multi-variate
# normal distribution, N(0,K), at all the positions in x
y = np.random.multivariate_normal(np.zeros(len(x)),K)
[/code]

The <code>multivariate_normal</code> function takes two arguments: (1) an array of noiseless mean values for each of the x-positions, and (2) a covariance matrix for all the x-positions.

In this case, I've made all of the  mean values equal to zero.

At the moment we haven't specified <strong>K</strong>, so these lines of code won't work just yet. To create <strong>K</strong> we need to build a matrix of values that are calculated by the function little <strong>k</strong>. This function is known as the <strong>covariance kernel</strong> and it defines how much of an affect one data value has on another.
<h3>The Covariance Kernel</h3>
To start with we are going to define a <em>squared-exponential covariance kernel</em>. This has the form:
<p style="text-align:center;">$latex k(x_n,x_m) = h^2 \exp{ \left( \frac{-(x_n - x_m)^2}{\lambda^2} \right)}
$</p>
where $latex x_n$ is the x-position of one data point and $latex x_m $ is the x-position of another. The value of the kernel is a function of how far away from each other these data points are, i.e. $latex x_n - x_m $. It also has a couple of <em>hyper-</em>parameters that govern the overall shape of the kernel: $latex h$ and $latex \lambda$. These are referred to as <strong>hyper-parameters</strong> because they don't really have any direct physical meaning so they're not bog-standard normal model parameters. Here the $latex h $ parameter controls the <em>normalisation</em> of the kernel and $latex \lambda $ controls the <em>width</em> of the kernel.

[code language="python" gutter="false" highlight="1-100"]
def cov_kernel(x1,x2,h,lam):

    """
    Squared-Exponential covariance kernel
    """

    k12 = h**2*np.exp(-1.*(x1 - x2)**2/lam**2)
    return k12
[/code]

We can then use this kernel function to fill our covariance matrix:

[code language="python" gutter="false" highlight="1-100"]
def make_K(x, h, lam):

    """
    Make covariance matrix from covariance kernel
    """

    # for a data array of length x, make a covariance matrix x*x:
    K = np.zeros((len(x),len(x)))

    for i in range(0,len(x)):
        for j in range(0,len(x)):

            # calculate value of K for each separation:
            K[i,j] = cov_kernel(x[i],x[j],h,lam)

 return K
 [/code]

and we can then update our earlier call to the numpy <code>multivariate_normal</code> function:

[code language="python" gutter="false" highlight="1-100"]
# make an array of 200 evenly spaced positions between 0 and 20:
x = np.arange(0, 20.,0.01) 

# make a covariance matrix:
K = make_K(x,h,lam)

# draw samples from a co-variate Gaussian
# distribution, N(0,K), at positions x1:
y = np.random.multivariate_normal(np.zeros(len(x)),K)
[/code]
<h3>Putting It All Together</h3>
We can expand this to look at what happens when we vary the hyper-parameters of the covariance kernel, in particular the width, $latex \lambda $:

[code language="python" gutter="false" highlight="1-100"]
# make an array of 200 evenly spaced positions between 0 and 20:
x = np.arange(0, 20.,0.01)

for i in range(0,3):

    h = 1.0

    if (i==0): lam = 0.1
    if (i==1): lam = 1.0
    if (i==2): lam = 5.0

    # make a covariance matrix:
    K = make_K(x,h,lam)

    # five realisations:
    for j in range(0,5):

        # draw samples from a co-variate Gaussian distribution, N(0,K):
        y = np.random.multivariate_normal(np.zeros(len(x)),K)

        tmp2 = '23'+str(i+3+1)
        pl.subplot(int(tmp2))
        pl.plot(x,y)

    tmp1 = '23'+str(i+1)
    pl.subplot(int(tmp1))
    pl.imshow(K)
    pl.title(r"$\lambda = $"+str(lam))

pl.show()
[/code]

<img class=" size-full wp-image-1878 aligncenter" src="https://allofyourbases.files.wordpress.com/2017/08/figure_1.png" alt="figure_1" width="873" height="670" />

The top row of plots is showing the covariance matrix for each of the three different kernel widths. The bottom row is showing five realisations of data <em>y</em>-values that correspond to each of the covariance matrices. Remember that all of these realisations have <strong>zero mean</strong> so the only contribution to the <em>y</em>-values is (covariate) Gaussian noise.

The narrower the covariance kernel, the more diagonal the covariance matrix becomes and the more like independent noise the <em>y</em>-values appear.
