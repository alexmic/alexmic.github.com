---
layout: post
title: "Which letter consumes the most pixels on screen?"
tldr: "It's M, but do read on."
published: true
---

I recently discovered an interesting [question](http://stackoverflow.com/questions/3949422/which-letter-of-english-alphabet-takes-up-most-pixels) on Stack Overflow:

> I am trying to do some dynamic programming based on the number of chars in a sentence. I need to know which letter of the English alphabet takes up the most pixels in the screen???

The most popular answer was great – simple and to the point. But, counting the *background* as pixels consumed by the letter didn't feel right to me so I wrote a little script to find the actual number. You can look at the code and results [here](https://gist.github.com/alexmic/8345076).

### Counting Helvetica pixels

Here's the whole script below. It uses `Pillow` to draw black letters on a white background and then count them. The top 3 letters are: **M** (2493 pixels), **W** (2414) and **B** (1909).


{% highlight python linenos=table %}
{% raw %}
from operator import itemgetter
from PIL import Image, ImageDraw, ImageFont


# Make a lowercase + uppercase alphabet.
alphabet = 'abcdefghijklmnopqrstuvwxyz'
alphabet += ''.join(map(str.upper, alphabet))


# We'll use Helvetica in big type.
helvetica = ImageFont.truetype('Helvetica.ttf', 100)


def draw_letter(letter, save=True):
    img = Image.new('RGB', (100, 100), 'white')

    draw = ImageDraw.Draw(img)
    draw.text((0,0), letter, font=helvetica, fill='#000000')

    if save:
        img.save("imgs/{}.png".format(letter), 'PNG')

    return img


def count_black_pixels(img):
    pixels = list(img.getdata())
    return len(filter(lambda rgb: sum(rgb) == 0, pixels))


if __name__ == '__main__':
    counts = [
        (letter, count_black_pixels(draw_letter(letter)))
        for letter in alphabet
    ]

    print sorted(counts, key=itemgetter(1), reverse=True)
{% endraw %}
{% endhighlight %}

### What about other fonts?

Let's now generalize this to a number of fonts since pixel count should vary between different font families. I'm on a Mac so I used `Font Book` to export a collection of fonts and then modified my script to calculate the mean and standard deviation for each letter.

The results are more or less the same: **M** (2217.51 ± 945.19), **W** (2139.06 ± 945.29) and **B** (1841.38 ± 685.26).

{% highlight python linenos=table %}
{% raw %}
# -*- coding: utf-8 -*-
from __future__ import division
import os
from collections import defaultdict
from math import sqrt
from PIL import Image, ImageDraw, ImageFont


# Make a lowercase + uppercase alphabet.
alphabet = 'abcdefghijklmnopqrstuvwxyz'
alphabet += ''.join(map(str.upper, alphabet))


def draw_letter(letter, font, save=True):
    img = Image.new('RGB', (100, 100), 'white')

    draw = ImageDraw.Draw(img)
    draw.text((0,0), letter, font=font, fill='#000000')

    if save:
        img.save("imgs/{}.png".format(letter), 'PNG')

    return img


def count_black_pixels(img):
    pixels = list(img.getdata())
    return len(filter(lambda rgb: sum(rgb) == 0, pixels))


def available_fonts():
    fontdir = '/Users/alex/Desktop/English'
    for root, dirs, filenames in os.walk(fontdir):
        for name in filenames:
            path = os.path.join(root, name)
            try:
                yield ImageFont.truetype(path, 100)
            except IOError:
                pass


def letter_statistics(counts):
    for letter, counts in sorted(counts.iteritems()):
        n = len(counts)
        mean = sum(counts) / n
        sd = sqrt(sum((x - mean) ** 2 for x in counts) / n)
        yield letter, mean, sd


def main():
    counts = defaultdict(list)

    for letter in alphabet:
        for font in available_fonts():
            img = draw_letter(letter, font, save=False)
            count = count_black_pixels(img)
            counts[letter].append(count)

    for letter, mean, sd in letter_statistics(counts):
        print u"{0}: {1:.2f} ± {2:.2f}".format(letter, mean, sd)


if __name__ == '__main__':
    main()
{% endraw %}
{% endhighlight %}

That's all. Now on to more productive things.


