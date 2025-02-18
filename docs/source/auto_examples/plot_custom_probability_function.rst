
.. DO NOT EDIT.
.. THIS FILE WAS AUTOMATICALLY GENERATED BY SPHINX-GALLERY.
.. TO MAKE CHANGES, EDIT THE SOURCE PYTHON FILE:
.. "auto_examples\plot_custom_probability_function.py"
.. LINE NUMBERS ARE GIVEN BELOW.

.. only:: html

    .. note::
        :class: sphx-glr-download-link-note

        Click :ref:`here <sphx_glr_download_auto_examples_plot_custom_probability_function.py>`
        to download the full example code

.. rst-class:: sphx-glr-example-title

.. _sphx_glr_auto_examples_plot_custom_probability_function.py:


============================================
Amputing with a custom probability function
============================================

.. GENERATED FROM PYTHON SOURCE LINES 8-9

Create complete data.

.. GENERATED FROM PYTHON SOURCE LINES 9-15

.. code-block:: default


    import numpy as np

    n = 10000
    X = np.random.randn(n, 2)








.. GENERATED FROM PYTHON SOURCE LINES 16-17

Define custom probability function.

.. GENERATED FROM PYTHON SOURCE LINES 17-27

.. code-block:: default


    # purely for demonstrative type hints
    from pyampute import ArrayLike

    # Must produce values between 0 and 1
    def min_max_scale(X: ArrayLike) -> ArrayLike:
        X_abs = np.abs(X)
        return (X_abs - X_abs.min()) / (X_abs.max() - X_abs.min())









.. GENERATED FROM PYTHON SOURCE LINES 28-33

Define some patterns.
Include the custom score to probability function in whichever pattern(s) you desire.
Here we'll create 3 patterns.
Note that the first and last pattern have the same weights but use different ``score_to_probability_func`` s.
The first pattern introduces missingness to feature 0, and the latter two introduce missingness to feature 1.

.. GENERATED FROM PYTHON SOURCE LINES 33-52

.. code-block:: default


    my_incomplete_vars = [np.array([0]), np.array([1]), np.array([1])]
    my_freqs = np.array((0.3, 0.2, 0.5))
    my_weights = [np.array([4, 1]), np.array([0, 1]), np.array([4, 1])]
    my_score_to_probability_funcs = [min_max_scale, "sigmoid-right", "sigmoid-right"]
    my_prop = 0.3

    patterns = [
        {
            "incomplete_vars": incomplete_vars,
            "freq": freq,
            "weights": weights,
            "score_to_probability_func": score_to_probability_func,
        }
        for incomplete_vars, freq, weights, score_to_probability_func in zip(
            my_incomplete_vars, my_freqs, my_weights, my_score_to_probability_funcs
        )
    ]








.. GENERATED FROM PYTHON SOURCE LINES 53-54

Run ampute.

.. GENERATED FROM PYTHON SOURCE LINES 54-60

.. code-block:: default

    from pyampute import MultivariateAmputation

    ma = MultivariateAmputation(prop=my_prop, patterns=patterns)
    incomplete_data = ma.fit_transform(X)









.. GENERATED FROM PYTHON SOURCE LINES 61-62

We expect about 30% of rows to be missing values

.. GENERATED FROM PYTHON SOURCE LINES 62-66

.. code-block:: default


    np.isnan(incomplete_data).any(axis=1).mean() * 100






.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none


    28.24



.. GENERATED FROM PYTHON SOURCE LINES 67-72

.. code-block:: default

    from pyampute.exploration.md_patterns import mdPatterns

    mdp = mdPatterns()
    pattern = mdp.get_patterns(incomplete_data)




.. image-sg:: /auto_examples/images/sphx_glr_plot_custom_probability_function_001.png
   :alt: plot custom probability function
   :srcset: /auto_examples/images/sphx_glr_plot_custom_probability_function_001.png
   :class: sphx-glr-single-img





.. GENERATED FROM PYTHON SOURCE LINES 73-75

Plot probabilities per pattern against the weighted sum scores per pattern.
Note that Pattern 1 and Pattern 3 have the same weights.

.. GENERATED FROM PYTHON SOURCE LINES 75-97

.. code-block:: default


    import matplotlib.pyplot as plt

    fig, ax = plt.subplots(
        len(patterns), 1, constrained_layout=True, sharex=True, sharey=True
    )
    for pattern_idx in range(len(patterns)):
        ax[pattern_idx].scatter(
            ma.wss_per_pattern[pattern_idx], ma.probs_per_pattern[pattern_idx]
        )
        score_to_prob_func = patterns[pattern_idx]["score_to_probability_func"]
        name = (
            score_to_prob_func
            if isinstance(score_to_prob_func, str)
            else score_to_prob_func.__name__
        )
        ax[pattern_idx].set_title(f"Pattern {pattern_idx + 1} ({name})")
    # supxlabel requires matplotlib>=3.4.0
    fig.supxlabel("Weighted Sum Score")
    fig.supylabel("Probability")
    plt.show()




.. image-sg:: /auto_examples/images/sphx_glr_plot_custom_probability_function_002.png
   :alt: Pattern 1 (min_max_scale), Pattern 2 (sigmoid-right), Pattern 3 (sigmoid-right)
   :srcset: /auto_examples/images/sphx_glr_plot_custom_probability_function_002.png
   :class: sphx-glr-single-img





.. GENERATED FROM PYTHON SOURCE LINES 98-102

Cases when you might not achieve desired amount of missingness
==============================================================
Here we rerun the amputation process but with only one pattern,
and that pattern uses a custom ``score_to_probability_func``.

.. GENERATED FROM PYTHON SOURCE LINES 102-112

.. code-block:: default


    patterns = [
        {"incomplete_vars": [np.array([0])], "score_to_probability_func": min_max_scale}
    ]
    ma = MultivariateAmputation(prop=my_prop, patterns=patterns)
    incomplete_data = ma.fit_transform(X)

    mdp = mdPatterns()
    pattern = mdp.get_patterns(incomplete_data)




.. image-sg:: /auto_examples/images/sphx_glr_plot_custom_probability_function_003.png
   :alt: plot custom probability function
   :srcset: /auto_examples/images/sphx_glr_plot_custom_probability_function_003.png
   :class: sphx-glr-single-img





.. GENERATED FROM PYTHON SOURCE LINES 113-114

We expect about 30% of rows to be missing values.

.. GENERATED FROM PYTHON SOURCE LINES 114-117

.. code-block:: default


    np.isnan(incomplete_data).any(axis=1).mean() * 100





.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none


    20.52



.. GENERATED FROM PYTHON SOURCE LINES 118-135

We expected 30% of rows to be missing values but when we only have one
pattern with a custom ``score_to_probability_func`` we don't see that result.

**This is expected behavior**.
For the sigmoid functions, we use ``prop`` to influence the proportion
of missingness by shifting the sigmoid function accordingly.
However, for a given custom probability we cannot know ahead of time
how to adjust the function in order to produce the desired proportion
of missingness.
In the previous example, we achieved nearly 30% missingness due to the
second and third patterns using the sigmoid ``score_to_probability_func``.

If you would like to use a custom probability function is it your responsibility
to adjust the function to produce the desired amount of missingness.
You can calculate the expected proportion of missingness following the procedure in Appendix 2 of `Schouten et al. (2018)`_.

.. _`Schouten et al. (2018)`: https://www.tandfonline.com/doi/full/10.1080/00949655.2018.1491577


.. rst-class:: sphx-glr-timing

   **Total running time of the script:** ( 0 minutes  0.523 seconds)


.. _sphx_glr_download_auto_examples_plot_custom_probability_function.py:


.. only :: html

 .. container:: sphx-glr-footer
    :class: sphx-glr-footer-example



  .. container:: sphx-glr-download sphx-glr-download-python

     :download:`Download Python source code: plot_custom_probability_function.py <plot_custom_probability_function.py>`



  .. container:: sphx-glr-download sphx-glr-download-jupyter

     :download:`Download Jupyter notebook: plot_custom_probability_function.ipynb <plot_custom_probability_function.ipynb>`


.. only:: html

 .. rst-class:: sphx-glr-signature

    `Gallery generated by Sphinx-Gallery <https://sphinx-gallery.github.io>`_
