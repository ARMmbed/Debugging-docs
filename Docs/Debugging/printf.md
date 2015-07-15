#Debugging with pyOCD

This article uses BLE as an example, but the techniques presented here are not BLE-specific.

##Printf()

Programs typically use the printf() family to communicate something readable back to the user:

1. The printf() functions produce output according to a format string (containing format specifiers) and matching value arguments. 
2. The microcontroller's universal asynchronous receiver/transmitter (UART) console peripheral "feeds" output from ``printf()`` into the interface chip. 
3. The chip forwards the feed to the development host. 
4. This printf() traffic can be viewed with a terminal program running on the host. 

<span style="background-color:#E6E6E6; border:1px solid #000;display:block; height:100%; padding:10px">
**Tip:** The following examples use the CoolTerm serial port application to read the ``printf()`` output, but you can use any terminal program you want and expect similar results.
</br>
**Tip:** The UART protocol requires that the sender and receiver each maintain their own clocks and know the baud rate. mbed interface chips use the 9,600 baud rate and your terminal program should be set to that baud rate to intercept the communication.
</span>

``printf()`` doesn’t come free - it exerts some costs on our program:

* An additional 5-10K of flash memory use. Do note, however, that this is the cost of the first use of ``printf()`` in a program; further uses cost almost no additional memory.

* Each call to ``printf()`` takes a significant time for processing and execution: about 100,000 instructions, or 10 milliseconds, depending on the clock speed. This is only a baseline: ``printf()`` with formatting will cost even more. If your clock runs slowly (as most microcontrollers' clocks do) and your computational power is therefore lower, ``printf()`` can sometimes cost so much it’s actually used as a delay.

These two costs require that we use ``printf()`` judiciously. First, because there is limited code-space on the microcontroller's internal flash. Second, because it delays the program so much. Be particularly careful about using it in an event handler, which we expect to terminate within a few microseconds.

<span style="background-color:#E6E6E6; border:1px solid #000;display:block; height:100%; padding:10px">
*Note:** ``printf()`` doesn’t require that you tell it beforehand how many parameters it should expect; it can receive any number you throw at it. To do this, you need to provide a format string with format specifiers, followed by a matching number of arguments. For example, ``printf(“temp too high %d”, temp)``: the format string is “temp too high %d”, and the format specifier is %d. The last bit is the argument: temp. It matches the format specifier %d, which specifies an integer. You can learn more on [Wikipedia](http://en.wikipedia.org/wiki/Printf_format_string).
</span>

Using ``printf()`` on mbed requires including the ``stdio`` header:

```c

	#include <stdio.h>

	... some code ...

		printf("debug value %x\r\n", value);
```

Here's a very basic example. In the [URI Beacon program](../GettingStarted/URIBeacon.md), we've added ``printf()`` in three places (this is too much for a real-life program):

* After setting ``DEVICE_NAME``, we've added ``printf("Device name is %s\r\n", DEVICE_NAME);``

* After ``startAdvertisingUriBeaconConfig();`` we've added ``printf("started advertising \r\n")``;.

* After ``ble.waitForEvent();`` we've added ``printf("waiting \r\n");``.

This is the terminal output. Note that "waiting" is printed every time ``waitForEvent`` is triggered:

<span style="text-align:center; display:block;">
![](../Debugging/Images/TerminalOutput1.png)
</span>


##Printf() Macros

There are some nifty tricks you can do with ``printf()`` using macro-replacement by the pre-processor.

The general form for a simple macro definition is:

	#define MACRO_NAME value 
This associates with the **MACRO_NAME** whatever **value** appears between the first space after the **MACRO_NAME** and the end of the line. The value constitutes the body of the macro.

``printf()``s are very useful for debugging when looking for an explanation to a problem. Otherwise, it is nice to be able to disable many of them. We can use the ``#define`` directive to create parameterized macros that extend the basic ``printf()`` functionality. For example, macros can expand to printf()s when needed, but to empty statements under other conditions. 

The general form for defining a parameterized macro is:

	#define MACRO_NAME(param1, param2, ...)      
		{body-of-macro}

For example, it is often useful to categorise ``printf()`` statements by severity levels like ‘DEBUG’, ‘WARNING’ and ‘ERROR’. For this, we define levels of severity. Then, each time we compile or run the program, we specify which level we’d like to use. The level we specified is used by our macros in an ``if`` condition. That condition can control the format of the information the macro will print, or whether or not it will print anything at all. This gives us full control of the debug information presented to us every run.

Remember that ``printf()`` can take as many parameters as you throw at it. Macros support this functionality: they can be defined with ``...`` to mimic printf()’s behaviour. To learn more about using ``...`` in your code, read about [variadic macros on Wikipedia](http://en.wikipedia.org/wiki/Variadic_macro).

Here is an example:

```c
	
	-- within some header file named something like trace.h --
	enum {
		TRACE_LEVEL_DEBUG,
		TRACE_LEVEL_WARNING
	};
	/* each time we compile or run the program, 
	* we determine what the trace level is.
	* this parameter is available to the macros 
	* without being explicitly passed to them*/
	
	extern unsigned traceLevel; 

	...

	// Our first macro is printed if the trace level we selected 
	// is TRACE_LEVEL_DEBUG or above. 	
	// The traceLevel is used in the condition                                            
	// and the regular parameters are used in the action that follows the IF
	#define TRACE_DEBUG(formatstring, parameter1, parameter2, ...) \
		{ if (traceLevel >= TRACE_LEVEL_DEBUG) \
				{ printf("-D- " formatstring, __VA_ARGS__); } } 
	// this will include the parameters we passed above
	
	// we create a different macro for each trace level
	#define TRACE_WARNING(formatstring, parameter1, parameter2, ...) \
		{ if (traceLevel >= TRACE_LEVEL_WARNING) \
			{ printf("-W- " formatstring, __VA_ARGS__); } }
```

Here’s another example of macro-replacement that allows a formatted ``printf()``. Set ``#define MODULE_NAME "<YourModuleName>"`` before including the code below, and enjoy colourised ``printf()`` tagged with the module name that generated it:

```c



	#define LOG(x, ...) \
		{ printf("\x1b[34m%12.12s: \x1b[39m"x"\x1b[39;49m\r\n", \
		MODULE_NAME, ##__VA_ARGS__); fflush(stdout); }
	#define WARN(x, ...) \
		{ printf("\x1b[34m%12.12s: \x1b[33m"x"\x1b[39;49m\r\n", \
		MODULE_NAME, ##__VA_ARGS__); fflush(stdout); }

```

You can use ``ASSERT()`` to improve error reporting. It will use ``error()`` (a part of the mbed SDK that we reviewed earlier). ``error()`` not only flashes LEDs, it also puts the program into an infinite loop, preventing further operations. This will happen if the ``ASSERT()`` condition is evaluated as FALSE:

```c

	#define ASSERT(condition, ...)	{ \
		if (!(condition))	{ \
			error("Assert: " __VA_ARGS__); \
		} }

```


##Fast Circular Log Buffers Based on Printf()

When trying to capture logs from events that occur in rapid succession, using ``printf()`` may introduce unacceptable run-time latencies, which might alter the system's behaviour or destabilise it. But delays in ``printf()`` aren’t because of the cost of generating the messages. The biggest cause of delay with ``printf()`` is actually pushing the logs to the UART. So the obvious solution is not to avoid ``printf()``, but to avoid pushing the logs to the UART while the operation we're debugging is running.

To avoid pushing during the operation’s run, we use ``sprintf()`` to write the log messages into a ring buffer (we’ll explain what that is in the next paragraph). The buffer holds the debugging messages in memory until the system is idle. Only then will we perform the costly action of sending the information through the UART. In BLE, the system usually idles in ``main()`` while waiting for events, so we’ll use ``main()`` to transmit.

``sprintf()`` assumes a sequential buffer into which to write - it doesn’t wrap strings around the end of the available memory. That means we have to prevent overflows ourselves. We can do this by deciding that we only append to the tail of the ring buffer if the buffer is at least half empty. In other words, so long as the information already held by the buffer doesn’t exceed the half-way mark, we will add new information "behind" it. When we reach the half-way point, we wrap-around the excess information to the beginning (rather than the tail) of the buffer, creating the “ring” of a ring buffer. Half is an arbitrary decision; you can decide to let the buffer get three-quarters full or only a tenth full.

Here is an example implementation of a ring buffer. We’ve created our own version of a wrapping ``printf()`` using a macro called ``xprintf()``.  Debug messages accumulated using ``xprintf()`` can be read out circularly starting from ``ringBufferTail`` and wrapping around (``ringBufferTail`` + ``HALF_BUFFER_SIZE``). The first message would most likely be garbled because of an overwrite by the most recently appended message:

```c


	#define BUFFER_SIZE 512 /* You need to choose a suitable value here. */
	#define HALF_BUFFER_SIZE (BUFFER_SIZE >> 1)

	/* Here's one way of allocating the ring buffer. */
	char ringBuffer[BUFFER_SIZE]; 
	char *ringBufferStart = ringBuffer;
	char *ringBufferTail  = ringBuffer;

	void xprintf(const char *format, ...)
	{
		va_list args;
		va_start(args, format);
		size_t largestWritePossible = BUFFER_SIZE - (ringBufferTail - ringBufferStart);
		int    written = vsnprintf(ringBufferTail, largestWritePossible, format, args);
		va_end(args);

		if (written < 0) {
			/* do some error handling */
			return;
		}

		/*
		* vsnprintf() doesn't write more than 'largestWritePossible' bytes to the
		* ring buffer (including the terminating null byte '\0'). If the output is
		* truncated due to this limit, then the return value ('written') is the
		* number of characters (excluding the terminating null byte) which would
		* have been written to the final string if enough space had been available.
		*/

		if (written > largestWritePossible) {
			/* There are no easy solutions to tackle this. It may be easiest to enlarge
			* your BUFFER_SIZE to avoid this. */
			return; /* this is a poor short-cut; you may want to do something else.*/
		}

		ringBufferTail += written;

		/* Is it time to wrap around? */
		if (ringBufferTail > (ringBufferStart + HALF_BUFFER_SIZE)) {
			size_t overflow = ringBufferTail - (ringBufferStart + HALF_BUFFER_SIZE);
			memmove(ringBufferStart, ringBufferStart + HALF_BUFFER_SIZE, overflow);
			ringBufferTail = ringBufferStart + overflow;
		}
	}


```

______
Copyright © 2015 ARM Ltd. All rights reserved.
