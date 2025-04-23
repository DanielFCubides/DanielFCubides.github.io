---
title: "Neural Networks Explained: Flight Similarity Prediction"
date: 2025-04-22
author: "Daniel F Cubides"
description: "Learn how neural networks work through a real-world example of flight similarity prediction"
tags: ["machine learning", "neural networks", "python", "tutorial"]
categories: ["Data Science", "Programming"]
---

# Neural Networks Explained: Flight Similarity Prediction

Hey there, tech enthusiasts and curious minds! 👋 Today we're diving into the fascinating world of neural networks with a real example: predicting flight similarities! Don't worry if you're new to this - I'll break it down so it's as easy as ordering your favorite pizza.

## 🧠 Neural Networks: The Basics

Imagine your brain for a second. It's made up of billions of neurons that send signals to each other, helping you recognize patterns, make decisions, and learn new things. Artificial Neural Networks (ANNs) are inspired by this amazing biological design!

### What's a Neural Network, Really?

A neural network is like a team of digital neurons working together to solve problems. Here's how they work in simple terms:

1. **Input Layer**: This is where we feed our data (like flight details in our example)
2. **Hidden Layers**: These middle layers do the heavy lifting - processing the information
3. **Output Layer**: This gives us our answer (like "yes, these flights are similar" or "nope, totally different")

Each "neuron" in these layers takes information, applies a mathematical function to it, and passes the result to the next layer. It's kind of like a relay race of calculations!

### Why Are Neural Networks So Cool?

Neural networks are amazing because they can:
- Learn from examples (just like humans!)
- Find patterns that might be invisible to the human eye
- Make predictions about things they've never seen before
- Handle messy, real-world data

## 🔍 Let's Analyze Our Flight Similarity Code

Now, let's look at the code we have for predicting flight similarities! This model helps determine when two flights might be related or substitutable for each other.

### Feature Engineering: Making Sense of Flight Data

First, our code transforms raw flight data into useful features:

```python
def feature_engineering(df):
    # Make a copy to avoid modifying the original
    df_copy = df.copy()

    # Convert times to numerical values (minutes)
    df_copy['flight_1_minutes'] = df_copy['Flight_1_time'].apply(convert_time_to_minutes)
    df_copy['flight_2_minutes'] = df_copy['Flight_2_time'].apply(convert_time_to_minutes)

    # Create new features
    # Price difference between flights
    df_copy['price_diff'] = abs(df_copy['flight_1_price'] - df_copy['flight_2_price'])
    df_copy['price_ratio'] = df_copy['flight_1_price'] / df_copy['flight_2_price']

    # Time difference between flights
    df_copy['time_diff'] = abs(df_copy['flight_1_minutes'] - df_copy['flight_2_minutes'])

    # Route similarity features
    df_copy['same_origin'] = (df_copy['flight_1_from'] == df_copy['flight_2_from']).astype(int)
    df_copy['same_destination'] = (df_copy['flight_1_to'] == df_copy['flight_2_to']).astype(int)
    df_copy['reversed_route'] = ((df_copy['flight_1_from'] == df_copy['flight_2_to']) &
                                 (df_copy['flight_1_to'] == df_copy['flight_2_from'])).astype(int)

    # Features for connections
    df_copy['possible_connection'] = ((df_copy['flight_1_to'] == df_copy['flight_2_from']) &
                                      (df_copy['flight_1_minutes'] < df_copy['flight_2_minutes'])).astype(int)

    # Drop the original time columns as we now have numerical versions
    df_copy.drop(['Flight_1_time', 'Flight_2_time'], axis=1, inplace=True)

    return df_copy
```

This is like preparing ingredients before cooking! We're transforming raw flight data into something our neural network can digest.

### The Neural Network Architecture

Let's look at the heart of our code - the neural network model:

```python
def build_similarity_model(input_dim):
    """Build a neural network model for flight similarity prediction."""
    model = Sequential([
        Dense(64, activation='relu', input_dim=input_dim),
        Dropout(0.3),
        Dense(32, activation='relu'),
        Dropout(0.3),
        Dense(32, activation='relu'),
        Dropout(0.3),
        Dense(32, activation='relu'),
        Dropout(0.3),
        Dense(32, activation='relu'),
        Dropout(0.3),
        Dense(1, activation='sigmoid')  # Output between 0-1 for similarity
    ])

    model.compile(
        optimizer='adam',
        loss='binary_crossentropy',
        metrics=['accuracy', tf.keras.metrics.AUC(), tf.keras.metrics.Precision(), tf.keras.metrics.Recall()]
    )

    return model
```

This creates a neural network with:
- An input layer that takes in our flight features
- Five hidden layers with varying numbers of neurons
- "Dropout" layers that help prevent overfitting (more on this later!)
- An output layer that gives us a similarity score between 0 and 1

## 🤔 Why This Neural Network Works

Our flight similarity neural network has some smart design choices:

### Multiple Hidden Layers (Deep Learning)

The model uses five hidden layers, making it what we call a "deep" neural network. These multiple layers help the network learn increasingly complex patterns:
- Earlier layers might learn basic relationships like "similar departure times"
- Middle layers might combine these into patterns like "similar route segments" 
- Later layers can abstract even further to complex relationships

Think of it like this: If you were trying to spot similar flights, you'd first check basic things (same airports?), then look at more complex patterns (similar times AND prices?), and finally make a judgment combining everything. Each layer in our network does something similar!

### ReLU Activation: The Secret Sauce

Each hidden layer uses the ReLU (Rectified Linear Unit) activation function. It's like a bouncer at a club that says "if the value is negative, make it zero; otherwise, let it through." 

```python
# ReLU in simple terms:
def relu(x):
    return max(0, x)
```

This simple function helps the network learn faster and avoid the "vanishing gradient problem" that plagued older neural networks.

### Dropout Layers: Fighting Overfitting

Those `Dropout(0.3)` layers are super important! They randomly turn off 30% of the neurons during training, which:
- Forces the network not to rely too much on any single neuron
- Acts like training multiple smaller networks
- Improves generalization (how well it works on new data)

It's like studying for a test where you know some pages will be missing - you have to understand the whole subject, not just memorize specific facts!

### Sigmoid Output: Perfect for Similarity

The final layer uses a sigmoid activation function, which squishes any value to between 0 and 1:

```python
# Sigmoid in simple terms:
def sigmoid(x):
    return 1 / (1 + math.exp(-x))
```

This is perfect for similarity prediction since we want a percentage-like score:
- 0 means "completely different flights"
- 1 means "these flights are practically identical"

## 🎯 How Many Layers Should You Use?

The million-dollar question! This model uses five hidden layers, but is that the right number? Here's a simple guide:

### Layer Count Guidelines:

1. **Simple problems** (like linear classification): 1-2 hidden layers
2. **Medium complexity** (like image recognition): 3-5 hidden layers
3. **Complex problems** (like natural language processing): 5+ hidden layers

For our flight similarity model, five layers might be overkill for some datasets, but it gives the model plenty of capacity to learn complex relationships between flights.

### The Goldilocks Principle: Not Too Few, Not Too Many

Choosing the number of layers is like the story of Goldilocks - you want it just right:

- **Too few layers**: Your model might be too simple to capture complex patterns (underfitting)
- **Too many layers**: You might overfit to your training data or face vanishing gradient problems

In our code, we can see the model has a reasonable architecture:
- It starts with 64 neurons in the first hidden layer
- Then has four more layers with 32 neurons each
- Each layer is followed by dropout to prevent overfitting

This is a good approach for moderately complex problems like flight similarity!

### How to Choose in Practice

There's no magic formula, but these principles can help:
- **Start small** and add layers if the model isn't learning well enough
- **Watch for diminishing returns** - sometimes more layers just waste computing power
- **Consider your data volume** - deeper networks need more training data
- **Monitor for overfitting** - if your model does great on training data but poorly on new data, you might need fewer layers or more dropout

The code also uses some great techniques to help with this:

```python
callbacks = [
    tf.keras.callbacks.EarlyStopping(patience=10, restore_best_weights=True),
    tf.keras.callbacks.ReduceLROnPlateau(factor=0.2, patience=5)
]
```

The `EarlyStopping` callback stops training when the model stops improving, which helps prevent overfitting. And `ReduceLROnPlateau` reduces the learning rate when progress stalls, helping the model fine-tune its weights.

## 🚀 Practical Tips for Your Own Neural Networks

If you're building your own neural network for a similar problem, here are some fun tips:

1. **Start simple!** Begin with 1-2 hidden layers and see how it performs before adding complexity.

2. **Feature engineering is king!** Good features (like our flight comparison features) can make even a simple model perform well.

3. **Always use dropout** - it's like magic fairy dust that prevents overfitting. The 30% rate in our code is a good starting point.

4. **Monitor training curves** - if your training accuracy keeps improving but validation doesn't, you're probably overfitting.

5. **Experiment with layer sizes** - a common pattern is to start wider (more neurons) and get narrower as you go deeper, like our model does (64 → 32 → 32 → 32 → 32).

## Code Analysis: Alternative Architectures

Let's look at how we might create simpler or more complex versions of our model:

### A Simpler Model (2 Hidden Layers)

```python
def build_simpler_model(input_dim):
    """A simpler model with fewer layers."""
    model = Sequential([
        # First hidden layer: 64 neurons
        Dense(64, activation='relu', input_dim=input_dim),
        Dropout(0.3),
        
        # Second hidden layer: 32 neurons
        Dense(32, activation='relu'),
        Dropout(0.3),
        
        # Output layer
        Dense(1, activation='sigmoid')
    ])
    
    model.compile(
        optimizer='adam',
        loss='binary_crossentropy',
        metrics=['accuracy']
    )
    
    return model
```

When to use this simpler model:
- When you have a smaller dataset (fewer than 1000 examples)
- When the relationship between features and similarity is fairly straightforward
- When you need faster training times
- As a baseline to compare against more complex models

### A More Complex Model (7 Hidden Layers)

```python
def build_deeper_model(input_dim):
    """A deeper model with more layers."""
    model = Sequential([
        # First hidden layer
        Dense(128, activation='relu', input_dim=input_dim),
        Dropout(0.3),
        
        # More hidden layers with decreasing neuron counts
        Dense(64, activation='relu'),
        Dropout(0.3),
        
        Dense(64, activation='relu'),
        Dropout(0.3),
        
        Dense(32, activation='relu'),
        Dropout(0.3),
        
        Dense(32, activation='relu'),
        Dropout(0.3),
        
        Dense(16, activation='relu'),
        Dropout(0.3),
        
        Dense(16, activation='relu'),
        Dropout(0.3),
        
        # Output layer
        Dense(1, activation='sigmoid')
    ])
    
    model.compile(
        optimizer='adam',
        loss='binary_crossentropy',
        metrics=['accuracy']
    )
    
    return model
```

When to use this deeper model:
- With large datasets (10,000+ examples)
- When you have many features with complex interactions
- When simpler models aren't capturing the patterns well
- When you have the computational resources for longer training times

## 🎭 Conclusion

Neural networks might seem like magic, but they're really just clever mathematical models inspired by our own brains. The flight similarity prediction model we explored demonstrates many best practices:

- Thoughtful feature engineering
- Multiple hidden layers to learn complex patterns
- Dropout for regularization
- Appropriate activation functions

Whether you're predicting flight similarities, recognizing faces, or translating languages, these principles apply across the neural network universe!

Happy learning, and may your neural networks always converge! 🧠✨
