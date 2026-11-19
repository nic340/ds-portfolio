# 🧠 How Learning Rate Shapes the Journey of a Small CNN

*An experiment on finding the right learning rate using Fashion-MNIST*

## 🌸 Introduction: Does One Small Number Make a Big Difference?

Imagine teaching a student to solve a problem. If you give them very small steps, they may take a long time to understand it. If you push them too quickly, they might skip important details and make mistakes. Training a neural network works in a similar way.

I wanted to understand how much the learning rate affects a Convolutional Neural Network (CNN). Would changing this one setting significantly affect its performance? And could a learning rate that works well for one training run become unstable in another?

To investigate, I experimented with five learning rates while keeping the model and other training settings consistent.

## 🐾 What Is a Learning Rate?

A neural network learns by adjusting its weights to reduce its loss. Loss is a numerical measure of how far the model's predictions are from the correct answers. During training, the model calculates gradients, which indicate how its weights should change to reduce that loss.

The learning rate controls the size of each adjustment. A simplified update rule is:

$$
\theta_{\text{new}} = \theta_{\text{old}} - \eta \nabla L(\theta)
$$

Here, \(\theta\) represents the model’s weights, \(\eta\) is the learning rate, and \(\nabla L(\theta)\) represents the gradient of the loss.

Think of it as walking down a hill while trying to reach the lowest point. Very small steps may get you there eventually, but progress can be slow. Extremely large steps may cause you to overshoot the lowest point repeatedly. A suitable step size helps you move toward the bottom efficiently.

The learning rate is a hyperparameter: a setting chosen before training rather than a value automatically learned by the model. An epoch is one complete pass through the training data. The Adam optimizer uses gradient information and adaptive updates to adjust model weights. A validation set helps evaluate performance during development, while a separate test set is reserved for the final evaluation. A random seed controls sources of randomness so that experiments can be repeated under comparable conditions.

## 🔬 The Experiment

I used the Fashion-MNIST dataset, which contains grayscale images of clothing across ten categories. The experiment used a training subset of 10,000 images, a validation set of 6,000 images, and a separate test set.

The same small CNN architecture was used for every run:

- Convolutional layers with 32 and 64 channels
- A fully connected layer with 128 hidden units
- ReLU activation, Batch Normalization, and dropout of 0.25
- Adam optimizer, without weight decay or a learning-rate scheduler

The batch size was 64, and each model trained for 8 epochs. I compared five learning rates: \(0.00001\), \(0.0001\), \(0.001\), \(0.01\), and \(0.1\). Each setting was tested with three random seeds: 0, 1, and 2.

This allowed me to compare the learning rates while also observing how much results varied across random seeds.

**Reproducibility:** The experiment notebook and code are available in my [GitHub repository](https://github.com/nic340/finding-the-right-learning-rate).

## 📊 Results and Observations

![Learning curves](https://raw.githubusercontent.com/nic340/finding-the-right-learning-rate/main/fig1_learning_curves.png)

*Figure 1. Training loss, validation loss, and validation accuracy across eight epochs for each learning rate. Lines represent the mean of three seeds, while shaded regions show the minimum-to-maximum range. The 0.001 learning rate achieved the strongest overall learning curves, while 0.00001 improved more slowly and 0.1 performed inconsistently.*

The learning curves show that the learning rate strongly influenced training progress. The rate of 0.001 produced the strongest overall results. Meanwhile, 0.00001 improved more slowly, suggesting that its updates were too small for the limited eight-epoch training period.

The following table summarizes the validation results.

| Learning rate | Best validation accuracy (mean ± std) | Final validation loss (mean ± std) |
|---|---:|---:|
| 0.00001 | 81.93% ± 0.32 | 0.541 ± 0.002 |
| 0.0001 | 88.47% ± 0.14 | 0.323 ± 0.003 |
| **0.001** | **89.13% ± 0.26** | **0.298 ± 0.010** |
| 0.01 | 87.07% ± 0.72 | 0.368 ± 0.030 |
| 0.1 | 53.12% ± 30.28 | 1.215 ± 0.774 |

*Table 1. Mean and standard deviation of the best validation accuracy and final validation loss across three seeds. The learning rate of 0.001 achieved the highest mean validation accuracy and lowest final validation loss.*

One particularly interesting observation came from the learning rate of 0.1. Its results varied dramatically across seeds.

| Seed | Best validation accuracy | Final validation loss |
|---|---:|---:|
| 0 | 75.17% | 0.636 |
| 1 | 10.30% | 2.309 |
| 2 | 73.90% | 0.698 |

*Table 2. Individual results for the learning rate of 0.1. Seed 1 collapsed to approximately chance-level accuracy for a ten-class classification task, while seeds 0 and 2 achieved considerably better results.*

This showed me that a learning rate can be problematic even when some runs appear successful. Looking at only one training run could hide instability.

![Learning-rate sensitivity](https://raw.githubusercontent.com/nic340/finding-the-right-learning-rate/main/fig2_sensitivity.png)

*Figure 2. Final validation loss against the learning rate on a logarithmic scale. Points summarize the mean across three seeds, with variability represented by the standard deviation. The lowest mean loss occurred at 0.001.*

The curve illustrates a U-shaped relationship. Both extremely small and extremely large learning rates produced higher validation losses than the middle settings. This supports the idea that choosing a suitable learning rate is important for effective training.

After comparing the validation results, I evaluated the 0.001 learning-rate setting on the separate test set using seed 0. It achieved **88.17% test accuracy**, with a test loss of **0.326**. This is a single-run test result, not an average across seeds.

## 💭 Discussion: What Did I Learn?

The experiment showed that the learning rate affects not only how quickly a CNN learns but also how consistently it performs.

The smallest learning rate, 0.00001, produced lower accuracy because its updates were likely too small to reach a strong solution within eight epochs. Increasing the rate to 0.0001 improved performance considerably. The 0.001 setting performed best among the tested values, achieving the highest mean validation accuracy and lowest mean validation loss.

At 0.01, performance remained competitive but was slightly worse than at 0.001. At 0.1, training became highly inconsistent. One seed reached only 10.30% validation accuracy, showing how an overly aggressive learning rate can prevent reliable learning.

However, these findings have limitations. I used only three random seeds, a subset of the available training data, and eight epochs. I also tested only one CNN architecture and one optimizer. Different models, datasets, training durations, or optimizers may produce different results.

## 🐱 Key Takeaways

1. **Start with a reasonable learning rate.** For this experiment, 0.001 was the strongest setting among those tested with Adam.
2. **Compare multiple learning rates.** A rate that is too small can slow learning, while one that is too large can make training unstable.
3. **Repeat experiments using different seeds.** Multiple runs reveal variation that a single run may hide.
4. **Use a separate test set.** Validation results help compare settings, while the test set provides a final evaluation.

My biggest takeaway is that successful model training is not only about choosing an architecture. Small hyperparameter decisions can make a meaningful difference in the learning process.

## 📚 References

- Instructor-provided class notebook: `hyperparameters_lightning.ipynb` (training setup adapted from the course material).
- Kingma, D. P., & Ba, J. (2015). *Adam: A Method for Stochastic Optimization.* arXiv:1412.6980.
- Xiao, H., Rasul, K., & Vollgraf, R. (2017). *Fashion-MNIST: A Novel Image Dataset for Benchmarking Machine Learning Algorithms.* arXiv:1708.07747.
- [PyTorch Lightning Documentation](https://lightning.ai/docs/pytorch/stable/).

## 🤖 AI Assistance and Disclosure

The starter notebook and course materials were provided by the instructor. I used Claude to assist with parts of the experiment notebook and ChatGPT to help organize and review the written explanation. 
