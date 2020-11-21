Emscripten Compiler Frontend (emcc)
***********************************

The Emscripten Compiler Frontend ("emcc") is used to call the Emscripten compiler from the command line. It is effectively a drop-in replacement for a standard compiler like *gcc* or *clang*.


Command line syntax
===================

   emcc [options] file...

(Note that you will need "./emcc" if you want to run emcc from your current directory.)

The input file(s) can be either source code files that *Clang* can handle (C or C++), LLVM bitcode in binary form, or LLVM assembly files in human-readable form.


Arguments
---------

Most clang options will work, as will gcc options, for example:       

   # Display this information
   emcc --help

   Display compiler version information
   emcc --version

To see the full list of *Clang* options supported on the version of *Clang* used by Emscripten, run "clang --help".

Options that are modified or new in *emcc* are listed below:

"-O0"
   No optimizations (default). This is the recommended setting for starting to port a project, as it includes various assertions.

   This and other optimization settings are meaningful both during compile and during link. During compile it affects LLVM optimizations, and during link it affects final optimization of the code in Binaryen as well as optimization of the JS. (For fast incremental builds "-O0" is best, while for release you should link with something higher.)

"-O1"
   Simple optimizations. During the compile step these include LLVM "-O1" optimizations. During the link step this does not include various runtime assertions in JS that *-O0* would do.

"-O2"
   Like "-O1", but enables more optimizations. During link this will also enable various JavaScript optimizations.

   Note:

     These JavaScript optimizations can reduce code size by removing things that the compiler does not see being used, in particular, parts of the runtime may be stripped if they are not exported on the "Module" object. The compiler is aware of code in --pre-js and --post-js, so you can safely use the runtime from there. Alternatively, you can use "EXTRA_EXPORTED_RUNTIME_METHODS", see src/settings.js.

"-O3"
   Like "-O2", but with additional optimizations that may take longer to run.

   Note:

     This is a good setting for a release build.

"-Os"
   Like "-O3", but focuses more on code size (and may make tradeoffs with speed). This can affect both wasm and JavaScript.

"-Oz"
   Like "-Os", but reduces code size even further, and may take longer to run. This can affect both wasm and JavaScript.

   Note:

     For more tips on optimizing your code, see Optimizing Code.

"-s OPTION[=VALUE]"
   Emscripten build options. For the available options, see src/settings.js.

   Note:

     You can prefix boolean options with "NO_" to reverse them. For example, "-s EXIT_RUNTIME=1" is the same as "-s NO_EXIT_RUNTIME=0".

   Note:

     If no value is specifed it will default to "1".

   Note:

     For options that are lists, you need quotation marks (") around the list in most shells (to avoid errors being raised). Two examples are shown below:

        -s RUNTIME_LINKED_LIBS="['liblib.so']"
        -s "RUNTIME_LINKED_LIBS=['liblib.so']"

   You can also specify that the value of an option will be read from a specified JSON-formatted file. For example, the following option sets the "EXPORTED_FUNCTIONS" option with the contents of the file at **path/to/file**.

      -s EXPORTED_FUNCTIONS=@/path/to/file

   Note:

     * In this case the file might contain a JSON-formatted list of functions: "["_func1", "func2"]".

     * The specified file path must be absolute, not relative.

   Note:

     Options can be specified as a single argument without a space between the "-s" and option name.  e.g. "-sFOO=1".

"-g"
   Preserve debug information.

   * When compiling to object files, this is the same as in *Clang* and *gcc*, it adds debug information to the object files.

   * When linking, this is equivalent to -g3.

"-gseparate-dwarf[=FILENAME]"
   Preserve debug information, but in a separate file on the side. This is the same as "-g", but the main file will contain no debug info. Instead, debug info will be present in a file on the side, in "FILENAME" if provided, otherwise the same as the wasm file but with suffix ".debug.wasm". While the main file contains no debug info, it does contain a URL to where the debug file is, so that devtools can find it. You can use "-s SEPARATE_DWARF_URL=URL" to customize that location (this is useful if you want to host it on a different server, for example).

"-g<level>"
   Controls the level of debuggability. Each level builds on the previous one:

      * "-g0": Make no effort to keep code debuggable.

      * "-g1": When linking, preserve whitespace in JavaScript.

      * "-g2": When linking, preserve function names in compiled code.

      * "-g3": When compiling to object files, keep debug info, including JS whitespace, function names, and LLVM debug info if any (this is the same as -g).

      * "-g4": When linking, generate a source map using LLVM debug information (which must be present in object files, i.e., they should have been compiled with "-g").

        Note:

          * Source maps allow you to view and debug the *C/C++ source code* in your browser's debugger!

          * This debugging level may make compilation significantly slower (this is why we only do it on "-g4").

"--profiling"
   Use reasonable defaults when emitting JavaScript to make the build readable but still useful for profiling. This sets "-g2" (preserve whitespace and function names) and may also enable optimizations that affect performance and otherwise might not be performed in "-g2".

"--profiling-funcs"
   Preserve function names in profiling, but otherwise minify whitespace and names as we normally do in optimized builds. This is useful if you want to look at profiler results based on function names, but do *not* intend to read the emitted code.

"--tracing"
   Enable the Emscripten Tracing API.

"--emit-symbol-map"
   Save a map file between the minified global names and the original function names. This allows you, for example, to reconstruct meaningful stack traces.

   Note:

     This is only relevant when *minifying* global names, which happens in "-O2" and above, and when no "-g" option was specified to prevent minification.

"--llvm-opts <level>"
   Enables LLVM optimizations, relevant when we call the LLVM optimizer (which is done when building source files to object files / bitcode). Possible "level" values are:

      * "0": No LLVM optimizations (default in -O0).

      * "1": LLVM "-O1" optimizations (default in -O1).

      * "2": LLVM "-O2" optimizations.

      * "3": LLVM "-O3" optimizations (default in -O2+).

   You can also specify arbitrary LLVM options, e.g.:

      --llvm-opts "['-O3', '-somethingelse']"

   You normally don't need to specify this option, as "-O" with an optimization level will set a good value.

"-flto"
   Enables link-time optimizations (LTO).

"--closure <on>"
   Runs the *Closure Compiler*. Possible "on" values are:

      * "0": No closure compiler (default in "-O2" and below).

      * "1": Run closure compiler. This greatly reduces the size of the support JavaScript code (everything but the WebAssembly or asm.js). Note that this increases compile time significantly.

      * "2": Run closure compiler on *all* the emitted code, even on **asm.js** output in **asm.js** mode. This can further reduce code size, but does prevent a significant amount of **asm.js** optimizations, so it is not recommended unless you want to reduce code size at all costs.

   Note:

     * Consider using "-s MODULARIZE=1" when using closure, as it minifies globals to names that might conflict with others in the global scope. "MODULARIZE" puts all the output into a function (see "src/settings.js").

     * Closure will minify the name of *Module* itself, by default! Using "MODULARIZE" will solve that as well. Another solution is to make sure a global variable called *Module* already exists before the closure-compiled code runs, because then it will reuse that variable.

     * If closure compiler hits an out-of-memory, try adjusting "JAVA_HEAP_SIZE" in the environment (for example, to 4096m for 4GB).

     * Closure is only run if JavaScript opts are being done ("-O2" or above).

"--pre-js <file>"
   Specify a file whose contents are added before the emitted code and optimized together with it. Note that this might not literally be the very first thing in the JS output, for example if "MODULARIZE" is used (see "src/settings.js"). If you want that, you can just prepend to the output from emscripten; the benefit of "--pre-js" is that it optimizes the code with the rest of the emscripten output, which allows better dead code elimination and minification, and it should only be used for that purpose. In particular, "--pre-js" code should not alter the main output from emscripten in ways that could confuse the optimizer, such as using "--pre-js" + "--post-js" to put all the output in an inner  function scope (see "MODULARIZE" for that).

   *--pre-js* (but not *--post-js*) is also useful for specifying things on the "Module" object, as it appears before the JS looks at "Module" (for example, you can define "Module['print']" there).

"--post-js <file>"
   Like "--pre-js", but emits a file *after* the emitted code.

"--extern-pre-js <file>"
   Specify a file whose contents are prepended to the JavaScript output. This file is prepended to the final JavaScript output, *after* all other work has been done, including optimization, optional "MODULARIZE"-ation, instrumentation like "SAFE_HEAP", etc. This is the same as prepending this file after "emcc" finishes running, and is just a convenient way to do that. (For comparison, "--pre-js" and "--post-js" optimize the code together with everything else, keep it in the same scope if running *MODULARIZE*, etc.).

"--extern-post-js <file>"
   Like "--extern-pre-js", but appends to the end.

"--embed-file <file>"
   Specify a file (with path) to embed inside the generated JavaScript. The path is relative to the current directory at compile time. If a directory is passed here, its entire contents will be embedded.

   For example, if the command includes "--embed-file dir/file.dat", then "dir/file.dat" must exist relative to the directory where you run *emcc*.

   Note:

     Embedding files is much less efficient than preloading them. You should only use it for small files, in small numbers. Instead use "--preload-file", which emits efficient binary data.

   For more information about the "--embed-file" options, see Packaging Files.

"--preload-file <name>"
   Specify a file to preload before running the compiled code asynchronously. The path is relative to the current directory at compile time. If a directory is passed here, its entire contents will be embedded.

   Preloaded files are stored in **filename.data**, where **filename.html** is the main file you are compiling to. To run your code, you will need both the **.html** and the **.data**.

   Note:

     This option is similar to --embed-file, except that it is only relevant when generating HTML (it uses asynchronous binary *XHRs*), or JavaScript that will be used in a web page.

   *emcc* runs tools/file_packager.py to do the actual packaging of embedded and preloaded files. You can run the file packager yourself if you want (see Packaging using the file packager tool). You should then put the output of the file packager in an emcc "-- pre-js", so that it executes before your main compiled code.

   For more information about the "--preload-file" options, see Packaging Files.

"--exclude-file <name>"
   Files and directories to be excluded from --embed-file and --preload-file. Wildcards (*) are supported.

"--use-preload-plugins"
   Tells the file packager to run preload plugins on the files as they are loaded. This performs tasks like decoding images and audio using the browser's codecs.

"--shell-file <path>"
   The path name to a skeleton HTML file used when generating HTML output. The shell file used needs to have this token inside it: "{{{ SCRIPT }}}".

   Note:

     * See src/shell.html and src/shell_minimal.html for examples.

     * This argument is ignored if a target other than HTML is specified using the "-o" option.

"--source-map-base <base-url>"
   The URL for the location where WebAssembly source maps will be published. When this option is provided, the **.wasm** file is updated to have a "sourceMappingURL" section. The resulting URL will have format: "<base-url>" + "<wasm-file-name>" + ".map".

"--minify 0"
   Identical to "-g1".

"--js-transform <cmd>"
   Specifies a "<cmd>" to be called on the generated code before it is optimized. This lets you modify the JavaScript, for example adding or removing some code, in a way that those modifications will be optimized together with the generated code.

   "<cmd>" will be called with the file name of the generated code as a parameter. To modify the code, you can read the original data and then append to it or overwrite it with the modified data.

   "<cmd>" is interpreted as a space-separated list of arguments, for example, "<cmd>" of **python processor.py** will cause a Python script to be run.

"--bind"
   Compiles the source code using the Embind bindings to connect C/C++ and JavaScript.

"--ignore-dynamic-linking"
   Tells the compiler to ignore dynamic linking (the user will need to manually link to the shared libraries later on).

   Normally *emcc* will simply link in code from the dynamic library as though it were statically linked, which will fail if the same dynamic library is linked more than once. With this option, dynamic linking is ignored, which allows the build system to proceed without errors.

"--js-library <lib>"
   A JavaScript library to use in addition to those in Emscripten's core libraries (src/library_*).

"-v"
   Turns on verbose output.

   This will pass "-v" to *Clang*, and also enable "EMCC_DEBUG" to generate intermediate files for the compiler's various stages. It will also run Emscripten's internal sanity checks on the toolchain, etc.

   Tip:

     "emcc -v" is a useful tool for diagnosing errors. It works with or without other arguments.

"--cache"
   Sets the directory to use as the Emscripten cache. The Emscripten cache is used to store pre-built versions of "libc", "libcxx" and other libraries.

   If using this in combination with "--clear-cache", be sure to specify this argument first.

   The Emscripten cache defaults to "emscripten/cache" but can be overridden using the "EM_CACHE" environment variable or "CACHE" config setting.

"--clear-cache"
   Manually clears the cache of compiled Emscripten system libraries (libc++, libc++abi, libc).

   This is normally handled automatically, but if you update LLVM inplace (instead of having a different directory for a new version), the caching mechanism can get confused. Clearing the cache can fix weird problems related to cache incompatibilities, like *Clang* failing to link with library files. This also clears other cached data. After the cache is cleared, this process will exit.

   By default this will also clear any download ports since the ports directory is usually within the cache directory.

"--clear-ports"
   Manually clears the local copies of ports from the Emscripten Ports repos (sdl2, etc.). This also clears the cache, to remove their builds.

   You should only need to do this if a problem happens and you want all ports that you use to be downloaded and built from scratch. After this operation is complete, this process will exit.

"--show-ports"
   Shows the list of available projects in the Emscripten Ports repos. After this operation is complete, this process will exit.

"--memory-init-file <on>"
   Specifies whether to emit a separate memory initialization file.

      Note:

        Note that this is only relevant when *not* emitting wasm, as wasm embeds the memory init data in the wasm binary.

   Possible "on" values are:

      * "0": Do not emit a separate memory initialization file.
        Instead keep the static initialization inside the generated JavaScript as text. This is the default setting if compiling with -O0 or -O1 link-time optimization flags.

      * "1": Emit a separate memory initialization file in binary format. This is more efficient than storing it as text inside JavaScript, but does mean you have another file to publish. The binary file will also be loaded asynchronously, which means "main()" will not be called until the file is downloaded and applied; you cannot call any C functions until it arrives. This is the default setting when compiling with -O2 or higher.

        Note:

          The safest way to ensure that it is safe to call C functions (the initialisation file has loaded) is to call a notifier function from "main()".

        Note:

          If you assign a network request to "Module.memoryInitializerRequest" (before the script runs), then it will use that request instead of automatically starting a download for you. This is beneficial in that you can, in your HTML, fire off a request for the memory init file before the script actually arrives. For this to work, the network request should be an XMLHttpRequest with responseType set to "'arraybuffer'". (You can also put any other object here, all it must provide is a ".response" property containing an ArrayBuffer.)

"-Wwarn-absolute-paths"
   Enables warnings about the use of absolute paths in "-I" and "-L" command line directives. This is used to warn against unintentional use of absolute paths, which is sometimes dangerous when referring to nonportable local system headers.

"--proxy-to-worker"
   Runs the main application code in a worker, proxying events to it and output from it. If emitting HTML, this emits a **.html** file, and a separate **.js** file containing the JavaScript to be run in a worker. If emitting JavaScript, the target file name contains the part to be run on the main thread, while a second **.js** file with suffix ".worker.js" will contain the worker portion.

"--emrun"
   Enables the generated output to be aware of the emrun command line tool. This allows "stdout", "stderr" and "exit(returncode)" capture when running the generated application through *emrun*. (This enables *EXIT_RUNTIME=1*, allowing normal runtime exiting with return code passing.)

"--cpuprofiler"
   Embeds a simple CPU profiler onto the generated page. Use this to perform cursory interactive performance profiling.

"--memoryprofiler"
   Embeds a memory allocation tracker onto the generated page. Use this to profile the application usage of the Emscripten HEAP.

"--threadprofiler"
   Embeds a thread activity profiler onto the generated page. Use this to profile the application usage of pthreads when targeting multithreaded builds (-s USE_PTHREADS=1/2).

"--em-config"
   Specifies the location of the **.emscripten** configuration file. If not specified emscripten will search for ".emscripten" first in the emscripten directory itself, and then in the user's home directory ("~/.emscripten"). This can be overridden using the "EM_CONFIG" environment variable.

"--default-obj-ext .ext"
   Specifies the file suffix to generate if the location of a directory name is passed to the "-o" directive.

   For example, consider the following command, which will by default generate an output name **dir/a.o**. With "--default-obj-ext .ext" the generated file has the custom suffix *dir/a.ext*.

      emcc -c a.c -o dir/

"--valid-abspath path"
   Note an allowed absolute path, which we should not warn about (absolute include paths normally are warned about, since they may refer to the local system headers etc. which we need to avoid when cross-compiling).

"-o <target>"
   The "target" file name extension defines the output type to be
   generated:

      * <name> **.js** : JavaScript (+ separate **<name>.wasm** file if emitting WebAssembly). (default)

      * <name> **.mjs** : ES6 JavaScript module (+ separate **<name>.wasm** file if emitting WebAssembly).

      * <name> **.html** : HTML + separate JavaScript file (**<name>.js**; + separate **<name>.wasm** file if emitting WebAssembly).

      * <name> **.bc** : LLVM bitcode.

      * <name> **.o** : WebAssembly object file (unless -flto is used in which case it will be in LLVM bitcode format).

      * <name> **.wasm** : WebAssembly without JavaScript support code ("standalone wasm"; this enables "STANDALONE_WASM").

   Note:

     If "--memory-init-file" is used, a **.mem** file will be created in addition to the generated **.js** and/or **.html** file.

"-c"
   Tells *emcc* to generate LLVM bitcode (which can then be linked with other bitcode files), instead of compiling all the way to JavaScript.

"--output_eol windows|linux"
   Specifies the line ending to generate for the text files that are outputted. If "--output_eol windows" is passed, the final output files will have Windows rn line endings in them. With "--output_eol linux", the final generated files will be written with Unix n line endings.

"--cflags"
   Prints out the flags "emcc" would pass to "clang" to compile source code to object/bitcode form. You can use this to invoke clang yourself, and then run "emcc" on those outputs just for the final linking+conversion to JS.


Environment variables
=====================

*emcc* is affected by several environment variables, as listed below:

   * "EMMAKEN_JUST_CONFIGURE"

   * "EMMAKEN_CFLAGS"

   * "EMCC_DEBUG"

   * "EMCC_CLOSURE_ARGS" : arguments to be passed to *Closure
     Compiler*

Search for 'os.environ' in emcc.py to see how these are used. The most interesting is possibly "EMCC_DEBUG", which forces the compiler to dump its build and temporary files to a temporary directory where they can be reviewed.


------------------------------------------------------------------

emcc: supported targets: llvm bitcode, javascript, NOT elf (autoconf likes to see elf above to enable shared object support)