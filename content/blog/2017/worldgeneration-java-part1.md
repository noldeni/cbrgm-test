---
title: "Part I: Procedual Map Generation with Java"
date:  2017-07-03T16:03:58+01:00
draft: false
tags: [ "Java", "Worldgeneration", "Game Development", "Algorithms" ]
categories:
  - "Development"
description: "This article is about how to create a simple randomly generated map using Java for game development purposes. "
---

Today I must make you a confession. I play computer games. But instead of just enjoying it and seeing it as a kind of competition, I'm interested in the implementation of the features that make a game playable.

A special feature that is used in many games today is the creation of randomly generated content using algorithms. Whether it's randomly created items, landscaping objects like trees or rock formations, all are now used in modern games like No Man's Sky, Star Citizen, Minecraft and many more to give the player a constantly new player experience.

Using algorithms, entire universes can be created in game environments. Many of these algorithms are based on a very similar principle of creating a randomly generated heightmap. A heightmap is a black-and-white image in which the contrasts between black and white represent the height.

In this small series I would like to share with you my experiences that I have made while I have dealt with the subject.

This first article is about how to create a simple randomly generated map using Java. In the following posts I would also like to enter different algorithms which I have tried and share with you the result. Perhaps I will also develop an algorithm and report on progress. But now let's start with the basics!

## Visualizing a 2D World!

![Result](/img/blog/2017/worldgeneration-java-part1/output_01.gif)

The world we want to create will be two-dimensional. Since we want to develop object-oriented, the "Separation of Concerns Principle" applies, which means that we want to separate the responsibilities of our classes neatly.

The process of generating our map can be roughly divided into two steps: **First** we will create a two-dimensional array and fill it with numbers. This array contains our raw information for visualizing the map. In the **second** step, we will use rules to determine exactly what is to be created on a specific coordinate.

### Creating the WorldGenerator

First, I create an interface for our world generators. The interface makes it possible for us to change the implementation of the world generation at a later stage.

The Interface looks likes this:

```java
public interface WorldGenerator {

	public int[][] createWorld(int width, int height);

}
```

All WorldGenerator Classes have to implement this function. We will later implement our first simple random algorithm.

### Creating a Map Visualizer

Since we want to get a visual representation of our generated map, we need a kind of visualization class that takes our generated two-dimensional array and colors the individual coordinates by defined rules.

The result of our visualization will be a simple PNG-Image file, that can be scaled by pixel size.

My first attempt of the visualizer looks like this, we will refactor it later on:

```java
public class MapImage {

	private final int PIXEL_SCALE = 10;

	/**
	 * Creates a 2D PNG Image from a two dimensional array.
	 *
	 * @param array
	 */
	public void visualize(int[][] array) {
		createImage(array, "generatedMap");
	}

	/**
	 * Creates an amount of 2D PNG Images from a two dimensional array.
	 *
	 * @param array
	 */
	public void visualize(int[][] array, int amount) {
		for (int i = 0; i < amount; i++) {
			createImage(array, "generatedMap" + i);
		}
	}

	/**
	 * Creates an amount of 2D PNG Images from a two dimensional array.
	 *
	 * @param array
	 */
	public void visualize(int[][] array, String filename) {
		createImage(array, filename);
	}

	/**
	 * Private Method to create a Buffered Image and save the result in a file.
	 *
	 * @param array
	 * @param filename
	 */
	private void createImage(int[][] array, String filename) {

		System.out.println("Creating MapImage, please wait...");

		int IMAGE_HEIGHT = array.length * PIXEL_SCALE;
		int IMAGE_WIDTH = array[0].length * PIXEL_SCALE;

		System.out.println("Image Width: " + IMAGE_WIDTH + "px");
		System.out.println("Image Height: " + IMAGE_HEIGHT + "px");

		// Constructs a BufferedImage of one of the predefined image types.
		BufferedImage bufferedImage = new BufferedImage(IMAGE_WIDTH, IMAGE_HEIGHT, BufferedImage.TYPE_INT_RGB);
		// Create a graphics which can be used to draw into the buffered image
		Graphics2D g2d = bufferedImage.createGraphics();

		// fill all the image with white
		g2d.setColor(Color.white);
		g2d.fillRect(0, 0, IMAGE_WIDTH, IMAGE_HEIGHT);

		for (int x = 0; x < array.length; x++) {
			for (int y = 0; y < array[x].length; y++) {

          //Defining coloring rules for each value
          //You may also use enums with switch case here

				if (array[x][y] == 0) { // if value equals 0, fill with water
					g2d.setColor(Color.BLUE);
					g2d.fillRect(y * PIXEL_SCALE, x * PIXEL_SCALE, PIXEL_SCALE, PIXEL_SCALE);

				} else if (array[x][y] == 1) { // if value equals 1, fill with land
					//...
				}
			}
		}
		// Disposes of this graphics context and releases any system resources
		// that it is using.
		g2d.dispose();

		System.out.printf("Saving MapImage to Disk as %s.png ... \n", filename);
		// Save as PNG
		File file = new File(filename + ".png");
		try {
			ImageIO.write(bufferedImage, "png", file);
		} catch (IOException e) {
			e.printStackTrace();
		}

		System.out.println("Done! \n");
	}
}

```

This may look complicated at first, but let me explain this to you. The MapImage class simply offers it's clients different visualization options. All visualization options use the private method `createImage(int[][] array, String filename)`. This method takes a two dimensional array and iterates over each value. In this example I defined two rules, saying if a value equals 0, draw a blue square at x,y (simply a pixel scaled by variable `PIXEL_SCALE`) and if a value equals 1, draw a green square at x,y.

In the end the result (BufferedImage) will be written into a `.png` file, which is stored in the project directory.

### Implementing a simple Randomly Filled Map Algorithm

Our first map algorithm is not really an algorithm. It's more a demonstration of how everything will work together in our program.

Here's the code of the RandomPoint WorldGenerator

```java
public class RandomPoint implements WorldGenerator {

	@Override
	public int[][] createWorld(int width, int height) {
		int[][] map = new int[width][height];

		Random r = new Random();

		for(int x = 0; x<map.length; x++) {
			for(int y = 0; y < map[x].length; y++) {
				map[x][y] = r.nextInt(2); //Random value between 0 and 1;
			}
		}
		return map;
	}
}
```

## Start Generating!

The only thing missing is the startup of our Map Generation tool. First create an instance of `WorldGenerator` and generate a 2D-Array. After that give the array as an argument to the `MapImage` class to create the Image files.

My code looks like this:

```java
public class Startup {

	public static void main(String[] args) {
		WorldGenerator worldgen = new RandomPoint();
		MapImage mi = new MapImage();

		for (int i = 0; i < 5; i++) {
			int[][] array = worldgen.createWorld(100, 100);
			mi.visualize(array, "generatedMap" + i);
		}

	}
}
```

This will create 5 `PNG` Files named "generatedMapX" in our project folder. **Congratulations!** We made our first steps into map generation!

In the next part we will enhance our simple RandomPoint Algorithm to something that looks more like a real landscape then a chess field. See you soon!
