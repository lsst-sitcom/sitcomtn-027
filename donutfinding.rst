Data
====
We used the data from the November 2021 run of AuxTel. specifically we used images ``seq_num`` 501-554 for this analysis.
We first smooth out the image by running it through :py:func:`cv2.GaussianBlur`. Next we normalize the flux across the
donut. This is done by determining the ``normValue`` for the donut and a background ``skyValue``
the normalized image is then:

.. code-block:: py
    :name: normalization

    # We normalize and remove background sky values:
    normValue = np.percentile(cutoutSmoothed, normPercent)
    skyValue = np.percentile(cutoutSmoothed, skyPercent)

    normImage = (smoothedImage - skyValue)/normValue

The ``normPercent`` and ``skyPercent`` are dependent on the date the data was taken. 
Next we use the ``normImage`` for determining a mask that will pick out only the donut part of the image. We convert this
mask into a ``uint8`` copy of the image, this last part is for finding finding the inner and outer cirlces only. While the
mask is used both for this and for the gradient fit, see below. 

For obtaining the circles we use the :py:func:`cv2.HoughCircles` to get the circle centers and radii. 


Analyses
========

We analyzed the donut images using 2 different methods.

- Our first method was to estimate the inner and outer rings of the donut, and their centers. 
  If the image was perfectly in focus we would expect their center's to overlap. Since they are intentionally out of focus
  we obtain a lists of off-sets ``dx`` and ``dy``. These off-sets should be related to the position of the secondary mirror 
  or ``Hexapod`` positions. we expect the relation to be described by

  .. math:: 

      dx &= c_1 x_{hex} + c_2 y_{hex} + x_0\\
      dy &= c_3 x_{hex} + c_4 y_{hex} + y_0
  
  Hence we can do a fit and obtain the matrix **C** and the offset :math:`\vec{O}` from these we can then solve for the case that 
  ``dx`` and ``dy`` = 0: 

  .. math::

      \vec{r}_\text{focus} = {\bf C}^{-1} (-\vec{O})


- The second method is to instead look at the flux accross our donut. The flux is uneven because of the donut being in focus so we
  can fit the flux level accross the donut with a plane: 

  .. math:: F = \Delta_x x_ + \Delta_y y + f_0
  
  In turn we can then do a similar find a similar relation between the :math:`\vec{\Delta}` and the ``Hexapod`` postions as we could
  with our first method. Finally solving for when the gradient of the plane is zero. 


Code-base
---------
The code for used for the analysis is available in Stubbs laboratory groups github repo `PCWG-Auxtel <https://github.com/stubbslab/PCWG-AuxTel>`__ 
An example notebook of how to use the code is available in the technotes github repo.

Results
=======
The results for this analysis is 2 positions for the focus to be in. We can compare these to the reported focus postion from the 
**WaveFront sensor** which was found to be ``(-3.85, 2.26)`` for the night in question. 
From our two analyses we obtain ``(-3.70,2.35)`` and ``(-3.93, 2.23)`` respectively. 
So we have that the focus reported by **WFS** lies between the focuses we found, but very close. 
Hence we feel this shows that the **WFS** is indeed working as intended. 

.. figure:: /_static/gradarrows20211102-2.pdf
     :name: gradient arrows

     The result of the second method vs. the **WFS** result. the arrows are the estimated gradient of the flux, indicated by direction
     and size, for each of the images used in the analysis.