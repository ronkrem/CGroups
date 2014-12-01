<h1>Class-like access to C Modules</h1>

<i>Ron Kreymborg</i>

I was recently asked to add some additional features to a project I had collaborated on nearly five years ago. It was a quite complex system controller that monitored various temperatures and flows, controlled a couple of motors, and provided a menu based display with associated buttons and switches. It was written in C and contained about fifty code files along with their matching headers. It was neatly modularised with at least one code file for each device or peripheral. The additional features required stitching in some new capability to the motor controllers and some additional menu features.

It surprised me that it took several days of doing globalised name searches in my editor to become familiar again with which module was associated with what set of function names and how they were related to the many modules involved. Of late I have been taking advantage of the increasing number of capable free C++ compilers available, and that method of always dealing with a module via its class name got me thinking. If something like this technique could be mapped to a C project, the module that contains any particular function would be immediately obvious.

The method I chose to implement this is not new – in fact C++ itself uses a similar technique. A structure is defined that contains a set of function pointers to the matching functions that are public within the module. I call this function table definition a <b>GROUP</b>, although it is a simple structure definition. The Group header file is included with each module and contains the definition:

<pre>#define GROUP   typedef struct</pre>

A group type definition is declared in the module header file and is a list of that module’s public functions as pointers, using the same names and prototype definitions as in the code file. These functions are normally listed anyway in the header as <i>extern</i> public prototypes, only now they are pointers and in a structure definition. Part of the definition is a pointer to that group type. Here is a definition for an example module called Motor.c:

<pre>GROUP
{
   // Initialise the module data.
   void (*MotorInit)(void);
   // Start the motor rotating at the given speed and direction.
   int (*Start)(int speed, int direction);
   // Decelerate the motor to a stop.
   int (*Stop)(int decelRate);
} MotorGroup;
typedef const MotorGroup* MotorGroupPointer;
extern const MotorGroup GMotor;</pre>

The actual instance of this MotorGroup called GMotor is declared in the code file itself, typically after all the matching public functions have been declared so there is no need for forward declarations. I usually place it at the very end. For this example it would look like:

<pre>const MotorGroup GMotor =
{
   MotorInit,
   Start,
   Stop,
};</pre>

Note the <i>const</i> storage qualifiers used above. They ensure these constant data structures remain in flash memory and do not waste valuable RAM space. 

To access the Motor functions elsewhere in the project, the header file must be included and a pointer declared and assigned to the structure instance:

<pre>const MotorGroupPointer pMotor = &GMotor;</pre>

Now all the public functions within the Motor module can be called using this pointer. For example:

<pre>motorState = pMotor->Start(800, D_CW);</pre>

Of course the alternative calling technique using the structure name itself is just as valid and does not require the additional storage for the pointer.

<pre>motorState = GMotor.Start(800, D_CW);</pre>

While I prefer the syntax associated with using the pointer, this is just personal. 

Note that all functions and global variables in the module are declared <i>static</i>. There is no access to anything within the module except via the Group structure. For example, the MotorInit function implementation would begin:

<pre>static void MotorInit(void)
{</pre>

Note that the order of the function pointer declarations must exactly match the order defined in the instance declaration. There is no mechanism in standard compilers to pickup order mismatches here, so it is the programmer’s responsibility. 

This would normally be very dangerous. However, I have written an executable that is passed the path to a file containing a list of all directories associated with the project. It then examines each header file it finds for whether it contains a GROUP definition and if so, searches for the corresponding instance definition in the associated code file. It then verifies that the function names appear in matching order in both places. A call to this executable can be included in the normal project compile sequence, or just run from time to time.

1-12-2014

