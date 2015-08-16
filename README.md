# rtl\_power\_fftw

`rtl_power_fftw` is a program that obtains a power spectrum from RTL
devices using the FFTW library to do FFT.

It is inspired by the program `rtl_power` in `librtlsdr`.  However, the
said program has several deficiencies that limit its usage in
demanding environments, such as radio astronomy. I inspected
`rtl_power` hoping to modify it and obtain better performance, but
came to the conclusion that it would be an unfeasible
task. Measurements of FFT performance showed that the leading library
in the field of FFT - `fftw` - makes mincemeat of the routine used in
`rtl_power`, even on simple processors such as raspberryPi. Therefore,
the following requirements for a program to obtain power spectrum from
rtlsdr devices were set out:

  - the new program should use the `fftw` library
  - it will (of course) use `librtlsdr` to interface device
  - it should process spectra in a separate thread from data acquisition,
	to optimize observation time (continuous sampling)
  - it should be friendly to use and adhere to the UNIX philosophy of
	only doing one thing, but doing it well
  - the output of the program should be easy to further use with the 
	standard tools (like `gnuplot`).
  
This all lead to dropping the functionality to obtain spectra that are
wider than what can be sampled at once in a single execution of the
program; if you want to stitch spectra together, you can do it with a
wrapper and multiple calls to `rtl_power_fftw`. Desire to have simple
code to handle option parsing lead to the choice of TCLAP and
therefore C++. This further meant that to implement things in a neat
way, C++11 functionality snuck into the program.

This is the current state of what the program supports:

```
USAGE: 

   ./rtl_power_fftw  [-B <buffers>] [-b <bins in FFT spectrum>] [-c] [-d
                     <device index>] [-f <Hz>] [-g <1/10th of dB>] [-n
                     <repeats>] [-p <ppm>] [-r <samples/s>] [-s <bytes>]
                     [-t <seconds>] [--] [--version] [-h]

Where: 

   -B <buffers>,  --buffers <buffers>
     Number of read buffers (don't touch unless running out of memory).

   -b <bins in FFT spectrum>,  --bins <bins in FFT spectrum>
     Number of bins in FFT spectrum (must be even number)

   -c,  --continue
     Repeat the same measurement endlessly.

   -d <device index>,  --device <device index>
     RTL-SDR device index.

   -f <Hz>,  --freq <Hz>
     Center frequency of the receiver.

   -g <1/10th of dB>,  --gain <1/10th of dB>
     Receiver gain.

   -n <repeats>,  --repeats <repeats>
     Number of scans for averaging (incompatible with -t).

   -p <ppm>,  --ppm <ppm>
     Set custom ppm error in RTL-SDR device.

   -r <samples/s>,  --rate <samples/s>
     Sample rate of the receiver.

   -s <bytes>,  --buffer-size <bytes>
     Size of read buffers (leave it unless you know what you are doing).

   -t <seconds>,  --time <seconds>
     Integration time in seconds (incompatible with -n).

   --,  --ignore_rest
     Ignores the rest of the labeled arguments following this flag.

   --version
     Displays version information and exits.

   -h,  --help
     Displays usage information and exits.
```

Example use of `rtl_power_fftw` with `gnuplot` to draw a spectrum into
a png file:

    ./rtl_power_fftw -f 1420000000 -n 100 -b 512 |\
        gnuplot -e "set term png;plot '< cat -' u 2:3">plot.png

To compile the program, cd into the directory where you have cloned the code
and do:

    mkdir build
    cd build
    cmake ..
    make

This should make the `rtl_power_fftw` binary in the build directory.
If you copy it into a directory in your `PATH`, you can call it from everywhere.
The install bit of the cmake is still in the TODO phase...

##TODO:

  - cmake install/uninstall
  - ... (there's allways something, isn't it?!)

Many thanks to Andrej Lajovic for cleaning up the C++ code and implementing a
better buffer handling system.
