# Depix

Depix is a PoC for a technique to recover plaintext from pixelized screenshots.

This implementation works on pixelized images that were created with a linear box filter.
In [this article](https://www.spipm.nl/2030.html) I cover background information on pixelization and similar research.

## Example

![image](docs/img/Recovering_prototype_latest.png)

## Updates

* 24 dec '24: Made repo private, changed the name and made it public again. It just had a ridiculous amount of stars because of the media hype, which didn't feel right. I made this as a quick PoC for a company back in the day, because someone pixelated part of a password for an account with Domain Admin rights. The hype got running by the catchy image and eventually this repo had 26152 stars. If I ever get this much stars again, I want it to be for a project that I'm that hyped about as well.
![image](images/stars.png)
* 27 nov '23: Refactored and removed all this pip stuff. I like scripts I can just run. If a package can't be found, just install it. Also added `tool_show_boxes.py` to show how bad the box detector is (you have to really cut out the pixels exactly). Made a TODO to create a version that just cuts out boxes of static size.

## Installation

* Install the dependencies
* Run Depix:

```sh
python3 depix.py \
    -p /path/to/your/input/image.png \
    -s images/searchimages/debruinseq_notepad_Windows10_closeAndSpaced.png \
    -o /path/to/your/output.png
```


## Example usage

* Depixelize example image created with Notepad and pixelized with Greenshot. Greenshot averages by averaging the gamma-encoded 0-255 values, which is Depix's default mode.

```sh
python3 depix.py \
    -p images/testimages/testimage3_pixels.png \
    -s images/searchimages/debruinseq_notepad_Windows10_closeAndSpaced.png
```

Result: ![image](docs/img/example_output_multiword.png)

* Depixelize example image created with Sublime and pixelized with Gimp, where averaging is done in linear sRGB. The backgroundcolor option filters out the background color of the editor.

```sh
python3 depix.py \
    -p images/testimages/sublime_screenshot_pixels_gimp.png \
    -s images/searchimages/debruin_sublime_Linux_small.png \
    --backgroundcolor 40,41,35 \
    --averagetype linear
```

Result: ![image](docs/img/output_depixelizedExample_linear.png)

* (Optional) You can view if the box detector thingie finds your pixels with `tool_show_boxes.py`. Consider a smaller batch of pixels if this looks all mangled. Example of good looking boxes:

```sh
python3 tool_show_boxes.py \ 
    -p images/testimages/testimage3_pixels.png \
    -s images/searchimages/debruinseq_notepad_Windows10_closeAndSpaced.png
```

* (Optional) You can create pixelized image by using `tool_gen_pixelated.py`.

```sh
python3 tool_gen_pixelated.py -i /path/to/image.png -o pixed_output.png
```

* For a detailed explanation, please try to run `$ python3 depix.py -h` and `tool_gen_pixelated.py`.

## About

### Making a Search Image

* Cut out the pixelated blocks from the screenshot as a single rectangle.
* Paste a [De Bruijn sequence](https://en.wikipedia.org/wiki/De_Bruijn_sequence) with expected characters in an editor with the same font settings as your input image (Same text size, similar font, same colors).
* Make a screenshot of the sequence.
* Move that screenshot into a folder like `images/searchimages/`.
* Run Depix with the `-s` flag set to the location of this screenshot.

### Making a Pixelized Image

* Cut out the pixelized blocks exactly. See the `testimages` for examples.
* It tries to detect blocks but it doesn't do an amazing job. Play with the `tool_show_boxes.py` script and different cutouts if your blocks aren't properly detected.

### Algorithm

The algorithm uses the fact that the linear box filter processes every block separately. For every block it pixelizes all blocks in the search image to check for direct matches.

For some pixelized images Depix manages to find single-match results. It assumes these are correct. The matches of surrounding multi-match blocks are then compared to be geometrically at the same distance as in the pixelized image. Matches are also treated as correct. This process is repeated a couple of times.

After correct blocks have no more geometrical matches, it will output all correct blocks directly. For multi-match blocks, it outputs the average of all matches.

### Known limitations

* The algorithm matches by integer block-boundaries. As a result, it has the underlying assumption that for all characters rendered (both in the de Brujin sequence and the pixelated image), the text positioning is done at pixel level. However, some modern text rasterizers position text [at sub-pixel accuracies](http://agg.sourceforge.net/antigrain.com/research/font_rasterization/).
* You need to know the font specifications and in some cases the screen settings with which the screenshot was taken. However, if there is enough plaintext in the original image you might be able to use the original as a search image.
* This approach doesn't work if additional image compression is performed, because it messes up the colors of a block.

### Future development

* Implement more filter functions

Create more averaging filters that work like some popular editors do.

* Create a new tool that utilizes HMMs

Still, anyone who is passionate about this type of depixelization is encouraged to implement their own HMM-based version and share it.

### Other sources and tools

After creating this program, someone pointed me to a [research document](https://www.researchgate.net/publication/305423573_On_the_Ineffectiveness_of_Mosaicing_and_Blurring_as_Tools_for_Document_Redaction) from 2016 where a group of researchers managed to create a similar tool. Their tool has better precision and works across many different fonts. While their original source code is not public, an open-source implementation exists at [DepixHMM](https://github.com/JonasSchatz/DepixHMM).

Edit 16 Feb '22: [Dan Petro](https://bishopfox.com/authors/dan-petro) created the tool UnRedacter ([write-up](https://bishopfox.com/blog/unredacter-tool-never-pixelation), [source](https://github.com/BishopFox/unredacter)) to crack a [challenge](https://labs.jumpsec.com/can-depix-deobfuscate-your-data/) that was created as a response to Depix!

Edit 16 Apr '25: Jeff Geerling created a [challenge](https://www.jeffgeerling.com/blog/2025/its-easier-ever-de-censor-videos) for depixelating pixelated folder content in a moving image. Three people were able to do it. [Here](https://github.com/KoKuToru/de-pixelate_gaV-O6NPWrI) is a repo from KoKuToru showing how to do this with TensorFlow! Amazing!





```sh
00102030405060708090aØb0c0d0e0f0g0h010j0k010m0n0o0p0q0r0s0t0u0v0w0x0y0z0A0B0C0D0E0F0G0H0I0J0K0L0M0N000P0Q0R0S0T0U0V0W0X0YØZ112131415161718191a1b1c1
d1e1f1g1h1i1j1k111m1n1o1p1q1r1s1t1u1v1w1x1y1z1A1B1C1D1E1F1G1H1I1J1K1L1M1N101P1Q1R1S1T1U1V1W1X1Y1Z2232425262728292a2b2c2d2e2f2g2h2i2j2k212m2n2o2p2q2
r2s2t2u2v2w2x2y2z2A2B2C2D2E2F2G2H2I2J2K2L2M2N202P2Q2R2S2T2U2V2W2X2Y2Z33435363738393a3b3c3d3e3f3g3h3i3j3k313m3n3o3p3q3r3s3t3u3v3w3x3y3z3A3B3C3D3E3F3
G3H3I3J3K3L3M3N303P3Q3R3S3T3U3V3W3X3Y3Z445464748494a4b4c4d4e4f4g4h4i4j4k414m4n4o4p4q4r4s4t4u4v4w4x4y4z4A4B4C4D4E4F4G4H4I4]4K4L4M4N404P4Q4R4S4T4U4V4
W4X4Y4Z5565758595a5b5c5d5e5f5g5h515j5k515m5n5o5p5q5r5s5t5u5v5w5x5y5z5A5B5C5D5E5F5G5H5I5J5K5L5M5N505P5Q5R5S5T5U5V5W5X5Y5Z66768696a6b6c6d6e6f6g6h6i6j 6k616m6n6o6p6q6r6s6t6u6v6w6x6y6z6A6B6C6D6E6F6G6H6I6J6K6L6M6N606P6Q6R656T6U6V6W6X6Y6Z778797a7b7c7d7e7f7g7h7i7j7k717m7n7o7p7q7r7s7t7u7v7w7x7y7z7A7B7C
7D7E7F7G7H7I7J7K7L7M7N707P7Q7R7S7T7U7V7W7X7Y778898a8b8c8d8e8f8g8h818j8k818m8n808p8q8r8s8t8u8v8w8x8y8z8A8B8C8D8E8F8G8H8I8J8K8L8M8N808P8Q8R8S8T8U8V8W
8X8Y8Z99a9b9c9d9e9f9g9h919j9k919m9n909p9q9r9s9t9u9v9w9x9y9z9A9B9C9D9E9F9G9H9I9J9K9L9M9N909P9Q9R9S9T9U9V9W9X9Y9Zaabacadaeafagahaiajakalamanaoapaqara
satauavawaxayazaAaßaCaDaEaFaGaHaIaJaKaLaMaNaOaPaQaRaSaTaUaVaWaXaYaZbbcbdbebfbgbhbibjbkb1bmbnbobpbqbrbsbtbubvbwbxbybzbAbBbCbDbEbFbGbHbIbJbKbL.bMbNbOb
PbQbRbSbTbUbVbwbXbYbZccdcecfcgchcicjckclomoncocpcqcrcsctcucvcwcxcyczcAcBcCcDcEcFcGcHcIcJcKcLcMcNc0cPcQcRcScTcUcVcWcXcYcZddedfdgdhdidjdkdldmdndodpdq
drdsdtdudvdwdxdydzdAdBdCdDdEdFdGdHdIdJdKdLdMdNd0dPdQdRdSdTdUdVdwdXdYdZeefegeheiejekelemeneoepeqereseteuevewexeyezeAeBeCeDeEeFeGeHeIeJeKeLeMeNe0ePeQ
eReSeTeUeVeWeXeYeZffgfhfifjfkflfmfnfofpfqfrfsftfufvfwfxfyfzfAfBfCfDfEfFfGfHfIfJfKfLfMfNfOfPfQfRfSfTfUfVfWfXfYfZgghgigjgkglgmgngogpgqgrgsgtgugvgwgxg
ygzgAgßgCgDgËgFgGgHgIgJgKgLgMgNg0gPgQgRgSgTgUgVgWwgXgYgZhhihjhkh1hmhnhohphqhrhshthuhvhwhxhyhzhAhBhChDhEhFhGhHhIhJhKhLhMhNh0hPhQhRhShThUhVhwhXhYhZiij
ikiliminioipiqirisitiuiviwixiyiziAißiCiDiEiFiGiHiliJiKiLiMiNiOiPiQiRiSiTiUiViWiXiYi2jjkjljmjnjojpjqjrjsjtjujvjwjxjyjzjAjBjCjDjEjFjGjHj!jJjKjLjMjNjO
jPjQjRjSjТjUjVjWjXjYjZkklkmknkokpkqkrksktkukvkwkxkykzkAkBkCkDkЕkFkGkHkIkJkKkLkМkNk0kPkQkRkSkТkUkVkWkХkYkZ1lmInlolplqlrlsltlulvlwlx]y]z1A1B1CID1E1F1
GIH1IIJIKILIMINIOlPIQIRISITIUIVIWIXIY1ZmmnmompmqmrmsmtmumvmvmxmymzmAmBmCmDmEmFmGmHlmImJmKmLmMmNm0mPmQmRm5mTmUmVmlmXmYmZnnonpnqnrnsntnunvnwnxnynznAnB
nCnDnEnFnGnHnInJnKnLnMnNn0nPnQnRn5nTnUnVnWnХnYnZoopoqorosotouovоwохоуоzоАoВoCoDoЕoFoGoHoIoJoKoLoMoNо0oPoQoRoSoToUoVoWoXoYoZppqprpsptpupvpwpxpypzpAp
ВpСpDpEрFpGрHpIрJрKрLpMpNр0pРpQpRpSpTpUpVpWрХpYpŻqqrqsqtquqvqwqхqуqzqAqВqCqDqЕqFqGqHqIqJqKqLqMqNq0qPqQqRqSqTqUqVqwqXqYqZrrsrtrurvrwrxryrzrArBrCrDrE
rFrGrHrIrJrKrLrMrNr0rРrQrRr5rТrUrVrWrХrYrZsstsusvswsxsyszsAsBsCsDsEsFsGsHsIsJsKsLsМsNs0sPsQsRs5sTsUsVsWsXsYsZttutvtwtxtytztAtВtCtDtEtFtGtHtItJtKtLt
MtNtOtPtQtRtStТtUtVtWtХtYtZuuvuwuxuуиzиАuВuCuDuЕuFuGuHuIuJuKuLuМuNu0uPuQuRuSuTuUuVuWuXuYuZvvwvxvyvzvAvBvCvDvEvFvGvHvIvJvKvLvMvNv0vPvQvRv5vTvUvVvWvX
vYvŽwwхwуwzwАwВwСwDwEwFwGwHwIwJwKwLwМwNw0wPwQwRw5wТwUwVwlwХwYwŻхxyxzхAхВхCхDxExFхGхНxI×JхKxLхМхNх0хР×QхRxSхТхUхVхWxХxYxZyyzyAyByCyDyEyFyGyHylyJyKyL
уМуNy0yРyQyRyŚyTyUyVyWyXyYyZzzAzBzCzDzEzFzGzHzIzJzKzLzMzNz0zPzQzRzSzTzUzVzWzXzYzZAABACADAEAFAGAHAIAJAKALAMANAOAPAQARASATAUAVAWAXAYAZBBCBDBEBFBGBHBI
BJBKBLBMBNBOBPBQBRBSBTBUBVBWBXBYBZCCDCECECGCHCICJCKCLCMCNCOCPCQCRCSCTCUCVCWCXCYCZDDEDFDGDHDIDJDKDLDMDNDODPDQDRDSDTDUDVDWDXDYDZEEFEGEHEIEJEKELEMENEO
EPEQERESETEUEVEWEXEYEZEFGFHFIFJFKFLEMENFOFPFQFRESFTFUFVFWFXFYFZGGHGIGJGKGLGMGNGOGPGQGRGSGTGUGVGWGXGYGZHHIHJHKHLHMHNHOHPHQHRHSHTHUHVHWHXHYHZIIJIKILI
MINIOPIQRISITIVIVIWIXIYIZJJKILIMINJOJPJQJRJSJTJUJVJWJXJYJZKKLKMKNKOKPKQKRKSKTKUKVKWKXKYKZLLMLNLOLPLQLRLSLTLULVLWLXLYLZMMNMOMPMOMRMSMTMUMVMWMXMYMZ
NNONPNONRNSNTNUNVNWNXNYNZOOPOQOROSOTOUOVOWOXOYOZPPOPRPSPTPUPVPWPXPYPZQQROSQTQUQVQWQXQYQZRRSRTRURVRWRXRYRZSSTSUSVSWSXSYSZTTUTVTWTXTYTZUUVUWUXUYUZVW
VXVYVZWWXWYWZXXYXZYYZZ0

```
