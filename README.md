# TFMU

The personal collection of weird behaviors and cryptic bugs of Tensorflow that totally fucks me up.

-----

## 1. tf.one_hot

*Behavior* : `tf.one_hot()` will crash under certain circumstances if the `indices` parameter is a `tf.uint8` type tensor.

This is actually a bug, but the error message is so cryptic that I wasted a whole day on finding this.

Please refer to this issue: https://github.com/tensorflow/tensorflow/issues/26099

*How to resolve*: Cast the tensor to `tf.int32`.

-----

## 2. `==` and `!=` operator is not overloaded

**Behavior**: `Tensor == int` is not the same as `tf.equal(Tensor, int)`, but `>=`, `<=`, `>` and `<` is the same as their corresponding 

This is an old issue and can be traced all the way back to TF 1.1.0rc2.

Please refer to this issue:
https://github.com/tensorflow/tensorflow/issues/9359

**TL;DR**: Basically, the `Tensor` has to be used as dictionary keys (say, using `feed_dict`), thus the `==` and `!=` operator cannot be overloaded.

The inconsistent behavior is not a bug, *IT'S DESIGNED* this way.

To quote the original issue author `carlthome`:
> It's `String` comparisons in Java all over again. :(

**How to resolve**: Try not to use operator while doing logical operations. Use `tf.equal` etc instead.

-----

## 3. tf.image.decode_jpeg

**Behavior**: By default, the decoded image pixel value is not the same as `cv2.imdecode` and `scipy.image.imdecode`.

Please refer to this:
https://stackoverflow.com/questions/45195880/why-does-tensorflow-decode-jpeg-images-differently-from-scipy-imread

**TL;DR**: Tensorflow uses a faster decode method, trading off accuracy for higher performance. The method is not compliant with JPEG standard, which requires maximum 1 bit of different in a pixel component using different decode variants.

**How to resolve**: Add `dct_method='INTEGER_ACCURATE'` while using `tf.decode_jpeg`. Or even better, *DON'T USE TENSORFLOW FOR IMAGE PROCESSING AT ALL.*

-----

## 4. tf.image.resize_*

**Behavior**: By default, the image corners are not aligned in `tf.image.resize_*` functions.

This is an infamous one.

Please refer to this brilliant blog post by Oleksandr Savsunenko:
https://hackernoon.com/how-tensorflows-tf-image-resize-stole-60-days-of-my-life-aba5eb093f35

Also, this issue:
https://github.com/tensorflow/tensorflow/issues/6720
The *Deeplab's Four Alignment Rules* by `gpapan` is very helpful.

**How to resolve**: Make sure `align_corner=True` *ALL THE TIME* if you have to use `tf.image.resize_*` functions. Also, follow the rules above.
