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
AABACADAEAFAGAHAIAJAKALAMANAOAPAQARASATAUAVAWAXAYAZAaAbAcAdAeAfAgAhAiAjAkAlAmAnAoApAqArAsAtAuAvAwAxAyAzA0A1A2A3A4A5A6A7A8A9Bb
BcBdBeBfBgBhBiBjBkBlBmBnBoBpBqBrBsBtBuBvBwBxByBzB0B1B2B3B4B5B6B7B8B9CcCdCeCfCgChCiCjCkClCmCnCoCpCqCrCsCtCuCvCwCxCyCzC0C1C2C3C
4C5C6C7C8C9DdDeDfDgDhDiDjDkDlDmDnDoDpDqDrDsDtDuDvDwDxDyDzD0D1D2D3D4D5D6D7D8D9EeEfEgEhEiEjEkElEmEnEoEpEqErEsEtEuEvEwExEyEzE0E1
E2E3E4E5E6E7E8E9FfFgFhFiFjFkFlFmFnFoFpFqFrFsFtFuFvFwFxFyFzF0F1F2F3F4F5F6F7F8F9GgGhGiGjGkGlGmGnGoGpGqGrGsGtGuGvGwGxGyGzG0G1G2G
3G4G5G6G7G8G9HhHiHjHkHlHmHnHoHpHqHrHsHtHuHvHwHxHyHzH0H1H2H3H4H5H6H7H8H9IiIjIkIlImInIoIpIqIrIsItIuIvIwIxIyIzI0I1I2I3I4I5I6I7I8
I9JjJkJlJmJnJoJpJqJrJsJtJuJvJwJxJyJzJ0J1J2J3J4J5J6J7J8J9KkKlKmKnKoKpKqKrKsKtKuKvKwKxKyKzK0K1K2K3K4K5K6K7K8K9LlLmLnLoLpLqLrLsL
tLuLvLwLxLyLzL0L1L2L3L4L5L6L7L8L9MmMnMoMpMqMrMsMtMuMvMwMxMyMzM0M1M2M3M4M5M6M7M8M9NnNoNpNqNrNsNtNuNvNwNxNyNzN0N1N2N3N4N5N6N7N8
N9OoOpOqOrOsOtOuOvOwOxOyOzO0O1O2O3O4O5O6O7O8O9PpPqPrPsPtPuPvPwPxPyPzP0P1P2P3P4P5P6P7P8P9QqQrQsQtQuQvQwQxQyQzQ0Q1Q2Q3Q4Q5Q6Q7Q
8Q9RrRsRtRuRvRwRxRyRzR0R1R2R3R4R5R6R7R8R9SsStSuSvSwSxSySzS0S1S2S3S4S5S6S7S8S9TtTuTvTwTxTyTzT0T1T2T3T4T5T6T7T8T9UuUvUwUxUyUzU0
U1U2U3U4U5U6U7U8U9VvVwVxVyVzV0V1V2V3V4V5V6V7V8V9WwWxWyWzW0W1W2W3W4W5W6W7W8W9XxXyXzX0X1X2X3X4X5X6X7X8X9YyYzY0Y1Y2Y3Y4Y5Y6Y7Y8Y
9ZzZ0Z1Z2Z3Z4Z5Z6Z7Z8Z9aabacadaeafagahaja kalamananaoapaqarasatauavawaxayaz a0a1a2a3a4a5a6a7a8a9bbcbdbebfbgbhbibjbkblbmbn bob
pbqbrbsbtbubvbwbxbybz b0b1b2b3b4b5b6b7b8b9cccdcecfcgchcicjckclcmcncocpcqcrcsc t c u c v c w c x c y c z c0c1c2c3c4c5c6c7c8c9d
dedefdgdh didjdkdldmdndodpdqdrdsdtdudvdwdxdy dz d0d1d2d3d4d5d6d7d8d9eefeg eheiejekelemeneo ep eq er es et euev e w e x e y e
z e0e1e2e3e4e5e6e7e8e9fffgf h f i f j f k f l f m f n fo fp f q f r f s f t f u f v f w f x f y f z f0f1f2f3f4f5f6f7f8f9ggghg
 i g j g k g l g m g n go gp g q g r g s g t g u g v g w g x g y g z g0g1g2g3g4g5g6g7g8g9hhhi h j h k h l h m h n ho hp h q h
 r h s h t h u h v h w h x h y h z h0h1h2h3h4h5h6h7h8h9iiij i k i l i m i n io ip i q i r i s i t i u i v i w i x i y i z i0i
1i2i3i4i5i6i7i8i9jjjk j l j m j n jo jp j q j r j s j t j u j v j w j x j y j z j0j1j2j3j4j5j6j7j8j9kkkl k m k n ko kp k q k
r k s k t k u k v k w k x k y k z k0k1k2k3k4k5k6k7k8k9lllml n lo lp l q l r l s l t l u l v l w l x l y l z l0l1l2l3l4l5l6l7l
8l9mmm n m o m p m q m r m s m t m u m v m w m x m y m z m0m1m2m3m4m5m6m7m8m9nnn o npn qnr n s n t n u n v n w n x n y n z n0
n1n2n3n4n5n6n7n8n9ooo p oqo r o s o t o u o v o w o x o y o z o0o1o2o3o4o5o6o7o8o9pppq p r p s p t p u p v p w p x p y p z p0
p1p2p3p4p5p6p7p8p9qqqr q s q t q u q v q w q x q y q z q0q1q2q3q4q5q6q7q8q9rrrs r t r u r v r w r x r y r z r0r1r2r3r4r5r6r7r
8r9ssst s u s v s w s x s y s z s0s1s2s3s4s5s6s7s8s9tttu t v t w t x t y t z t0t1t2t3t4t5t6t7t8t9uuuv u w u x u y u z u0u1u2u
3u4u5u6u7u8u9vvvw v x v y v z v0v1v2v3v4v5v6v7v8v9www x w y w z w0w1w2w3w4w5w6w7w8w9xxx y x z x0x1x2x3x4x5x6x7x8x9yyy z y0y1y
2y3y4y5y6y7y8y9zzz yz0z1z2z3z4z5z6z7z8z90 01 02 03 04 05 06 07 08 091 12 13 14 15 16 17 18 192 23 24 25 26 27 28 293 34 35 36
 37 38 394 45 46 47 48 495 56 57 58 5960 61 62 6364 65 66 67 68 6970 71 72 7374 75 76 77 78 7980 81 8283 84 85 86 87 8890 91
9293 94 95 96 97 9899AA
```
