# Compiling Software (Basics)

Programs are often written in two types of languages: compiled and
interpreted. An interpreted language is read by a program (the
_intepreter_), and the interpreter carries out the corresponding actions,
such as doing calculations, etc. On the other hand, a compiled language
will first be converted into a binary executable by the _compiler_, then
it can directly run on the computer's CPU.

The bootloader, kernel, and nearly all Linux utilities are written in a
compiled language called `C`, because twenty years ago, when these things
were first written, `C` was the most versatile and reliable language
available. The bootloader and kernel couldn't be written in an interpreted
language, because the interpreter program itself isn't even loaded by the
time a bootloader is supposed to be running. You can read more about the
`C` language on Google.

The binary executables produced by the compilers are _platform dependent_.
That is, a binary file produced on one platform can't run on another
different platform. For example, if you compile a piece of C code on a
`x86-based PC`, then you can't run the resulting binary on a
`ARM-based Raspberry Pi`, since these two platforms have an incompatible
architecture, which means the instructions contained within the binaries
can only work for one of these platforms at a time.
