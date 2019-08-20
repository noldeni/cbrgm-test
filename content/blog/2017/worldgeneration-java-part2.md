---
title: "Part 2: Procedual Map Generation with Java"
date:  2017-08-11T16:03:58+01:00
draft: false
tags: [ "Java", "Worldgeneration", "Game Development", "Algorithms" ]
categories:
  - "Development"
description: "This article is about how to create a simple randomly generated map using Java for game development purposes. "
---

Welcome back!

Here comes part two of my procedual world generation with Java series. Today I'd like to introduce you to an generation algorithm called Simplex Noise, which is very common for terrain generation in games.

Simplex Noise? Never heard! That's okay. It was exactly the same as when I started to inform myself about the topic. The idea behind it is very simple. As a basis for our process-generated world, we need high-altitude data, which we assign to a terrain type in a second step according to their value. One way of generating these altitudes is to use a black-and-white image, each pixel being assigned a brightness value between 0 (black) to 255 (white). The brighter a pixel on the image, the higher this coordinate is in our generated world.

We do not want to color this map by hand, but this job is supposed to take over a so-called noise function for us. Noise functions create simple black-and-white images on which "fog" can be seen, similar to these images:

![Perlin Noise](https://thebookofshaders.com/11/simplex-noise.png)

## Implementing Simplex Noise

Ken Perlin is like the superstar of the noise function. At least, I often came across his name when I informed myself about the subject. He is the inventor of the original perlin noise algorithm, a noise function that is often used in computer graphics because of the very natural behavior used for partial effects, clouds or water. A little later in 2001, he published an improved version of the algorithm, the **Simplex Noise Algorithm**, which also generates random smooth gradients.

The Simplex Noise algorithm itself is not easy to understand and would break the boundaries here. If you are interested in this, you should consider Stefan Gustavson's paper (I linked it [here][143ff98c]), which has dealt intensively with the algorithm, much better then I could do. I will consider the implementation of the algorithm here as a black box. What is important to know is that it uses something known as a simple grid to add values ​​and values ​​between -1 and 1 that look linearly-interpolated like classic perline noise.

Simplex noise is a method for constructing a noise function comparable to Perlin noise but with fewer directional artifacts and, in higher dimensions, a lower computational overhead.

The Stefan Gustavson's Simplex Noise implementation can be found [here][bfb76b0c].

## Using Simplex Noise to create a 2D-Array

At first I created a `SimplexNoiseGenerator.class`, that is using the Algorithm to generate an array of height values. The Generator uses three major values, to create different results of generated output. The `octaves` value is explained below, `roughness` is used to increase the range between the -1 and 1 values which represents the terrain height (Increasing the value results in less flat landscapes) and `scale` is used as some kind of zoom factor.

```java
public SimplexNoiseGenerator(int octaves, double roughness, double scale) {
		this.OCTAVES = octaves; // Number of Layers combined together to get a natural looking surface
		this.ROUGHNESS = roughness; // Increasing the of the range between -1 and 1, causing higher values eg more
									// rough terrain
		this.SCALE = scale; // Overall scaling of the terrain
	}
```

The rest of my class implementation looks like this:

```java
/**
 * Overriding the method createWorld in class SimplexNoiseGenerator.
 * For further details have a look at:
 *
 * @see worldgeneration.WorldGenerator#createWorld(int, int)
 */
@Override
	public double[][] createWorld(int width, int height) {
		return generateOctavedSimplexNoise(width, height);
	}

	private double[][] generateOctavedSimplexNoise(int width, int height) {
		double[][] totalNoise = new double[width][height];
		double layerFrequency = SCALE;
		double layerWeight = 1;
		double weightSum = 0;

		// Summing up all octaves, the whole expression makes up a weighted average
		// computation where the noise with the lowest frequencies have the least effect

		for (int octave = 0; octave < OCTAVES; octave++) {
			// Calculate single layer/octave of simplex noise, then add it to total noise
			for (int x = 0; x < width; x++) {
				for (int y = 0; y < height; y++) {
					totalNoise[x][y] += SimplexNoise.noise(x * layerFrequency, y * layerFrequency) * layerWeight;
				}
			}

			// Increase variables with each incrementing octave
			layerFrequency *= 2;
			weightSum += layerWeight;
			layerWeight *= ROUGHNESS;

		}
		return totalNoise;
	}
```

You say that the above code is not easy to understand? Yes, thats true. The magic in the private method `generateOctavedSimplexNoise` is happening within the for-loops. Here the `octaves` value is important. Every octave represents an image layer that is generated. All layers (each octave) are stacked together in the end to create the final result. The more layers you stack together, the more detailed the landscape will become.

Here are example results I generated with the `octaves` value from 1-7 (try on your own!):

![Different Octaves](/img/blog/2017/worldgeneration-java-part2/output_01.gif)

## Adding a Seed to world

You might wonder why everytime you start the application, you get the same results. That's because we are using a noise function. A (mathematical) function takes arguments, and returns exact the same values every time you enter the same arguments again.A cool feature we can implement by using a function is the generation of "infinite" prodecual generated terrain.

I added a seed function to our `SimplexNoise.class` that rewrites the array used for noise generation based on a given number sequence.

This is the code I added to Stefan Gustavson's original `SimplexNoise.class`:

```java
public static void setSeed(long seed) {
		SimplexNoise.seed = seed;

		Random r = new Random(SimplexNoise.seed);

		for (int i = 0; i < p.length; i++) {
			p[i] = (short) r.nextInt(255);  
		}

		for (int i = 0; i < 512; i++) {
			perm[i] = p[i & 255];
			permMod12[i] = (short) (perm[i] % 12);
		}
	}
```

In the private method `generateOctavedSimplexNoise` of `SimplexNoiseGenerator.class` I am passing a random number to the static `setSeed` method (as seen above).

```java
private double[][] generateOctavedSimplexNoise(int width, int height) {
		double[][] totalNoise = new double[width][height];
		double layerFrequency = SCALE;
		double layerWeight = 1;
		double weightSum = 0;

		Random r = new Random();

		SimplexNoise.setSeed(r.nextInt(Integer.MAX_VALUE));
```

As a result we will receive different views of our "infinite" terrain generated by the Simplex Noise Algorithm and get a pseudo-random map generation

## Visualizing the world!

Finally we have to execute our application. I did not explain the `MapImage.class` in this part here. It simply takes a two-dimensional array of values and creates an image where each pixel is colored by rules. Please have a look at Part I of terrain generation that I wrote earlier on. The code below generates 10 `png` images, have a look:

```java
public class Startup {

	public static void main(String[] args) {

		WorldGenerator worldgen = new SimplexNoiseGenerator(7, 0.6f, 0.0050f);
		MapImage mi = new MapImage();

		for (int i = 0; i < 8; i++) {
			double[][] array = worldgen.createWorld(250, 250);
			mi.visualize(array, "generatedMap" + i);
		}
	}
}
```

![World Results](/img/blog/2017/worldgeneration-java-part2/output_02.gif)

I hope you enjoyed reading Part II of my Prodecual Map Generation with Java and try the code on your own. See you next time!

[143ff98c]: http://weber.itn.liu.se/~stegu/simplexnoise/simplexnoise.pdf "Simplex Noise"

[bfb76b0c]: http://staffwww.itn.liu.se/~stegu/simplexnoise/SimplexNoise.java "SimplexNoise.java"
