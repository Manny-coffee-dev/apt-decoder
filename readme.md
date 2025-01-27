apt-decoder
===========
A simple tool for decoding the Automatic Picture Transmissions broadcasted
by the **NOAA Weather satellites** to PNG files.

Existing tools like WxToImg or atp-dect are either not open source or no longer maintained
and unnecessarily complex.
apt-decoder provides a lightweight, simple to use and easy to understand solution.

![short sample](noaa19_short.png)


Building
--------
1. Install the rust compiler and cargo.
    E.g. using rustup (Try installing rustup using yourpackage manager,
    don't use the stupid **curl | sh** stuff.)
2. Run `cargo build --release`
3. The `apt-decoder` binary is in `target/release`
4. Done


Usage
-----
1. Save a received and FM-demodulated satellite signal as WAV-file.
    The WAV file has to be **mono**, **48kHz** and **32bit float**.
    When in doubt you can use audacity to convert your file into this format.
2. Run `apt-decoder <your WAV file> <destination PNG file>`
    You can also run the example contained in this repo:
    `apt-decoder noaa19_short.wav noaa19_short.png`
3. Look at the generated PNG file, adjust the dynamic and contrast with your favorite tool.
4. Done

![long sample](example.png)

Theory of Operation
-------------------
![flowgraph](flow.png)

The AM-data from the WAV files is decoded using a simple square law demodulator.
Squaring the input signals yields the square of the original signal
and sideband with the twice the carrier frequency.

Computing the square root of the signal does not change its frequency spectrum,
but restores the amplitude of the original signal part.
The square root function is usually frowned upon in literature about signal processing,
as it is rather complex to implement on traditional DSP architectures.
In contrast to that ALUs in modern x86 or ARM CPUs can computer the square root efficiently,
so there is no harm in taking a shortcut here.

A lowpass then gets rid of the higher sideband, leaving only the demodulated original signal.

Afterwards the signal is sampled down to 4160kHz,
as original signal contains 4160 pixels per line.
For additional efficiency the upsampler inserts zeros into the input signal instead
of actually interpolating the missing samples.
This works because the downsampler following afterwards works by computing the average
over a number of samples and therefore acts as a low pass.

Finally a line syncer module looks for the **sync A** and **sync B** patterns to add
line sync information to the signal, which can then be written out into a PNG file.

Questions
---------
Feel free to write emails to sebastian(a)sebastians-site.de

Alternatively you can try [twitter](https://twitter.com/l_h_hacker), [mastodon](https://chaos.social/@sebastian) or *sebastian* on *hackint*.
