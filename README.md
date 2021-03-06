# Introduction

Welcome to an introduction to the SuperCollider programming language!  The aim of this guide is to comprehensively walk the reader through the basics of writing code in SuperCollider (being careful to touch on syntactic semantics that might trip up new users), and then to explain how one can use SuperCollider to create musical instruments and generate algorithmic music.

### Assumptions

You'll need to know a couple of things coming into this guide. They are:
- The basics of coding (in some ordinary language – Java, C, Python, etc)
- What SuperCollider is and what it does

In short, take a look at [this FAQ](https://scacinto.wordpress.com/2014/02/24/intro-to-supercollider-3-for-the-uninitiated/#more-2318) before continuing on, if you're completely new. Download the IDE, open it up, and let's dive in!

# The Basics

### The Editor and Code Execution

You'll notice there are three parts to the IDE: the post window in the bottom right, the help browser in the top right, and the editor on the left.  We'll be primarily concerned with the editor, since that's where we write code!  Go ahead and copy the below into your editor:

```
'Hello, World!'.postln;
```

Click on the line and hit cmd + return to execute it –  you should see 'Hello, World!' pop up in the post window.  We "posted" it there with postln.  As in most programming languages, single or double quotes identify a string.  The following syntax also works:

```
postln('Hello, World!');
```

If you execute this line in the same way as above, you should see the same results.  SuperCollider is, in general, very syntactically flexible – this can be both a blessing and a curse.  You can pick the style that suits you best, but if you mix and match, your code will become messy and difficult to read!

You've probably also noticed that we're able to execute lines of code individually and immediately, rather than having to compile and run an entire program.  For those familiar, SuperCollider is similar to R in this respect.  We can execute multiple lines of code sequentially by surrounding a code block with parenthesis:

```
(
'This is a statement...'.postln;
'...And so is this!'.postln;
)
```

Clicking anywhere inside the parenthesis and hitting cmd + return will execute every statement in the block, one by one.  SuperCollider doesn't really care about whitespace, so long as you provide semicolons to break up individual statements.  So it's up to you to style your code properly!

You may have noticed that the post window prints '...And so is this!' an additional time.  Regardless of whether or not you're using postln, SuperCollider sends the output of the last statement in any executed code block to the post window.

### Variables

If you're already familiar with general programming practice, by this time you're probably wondering where our variables are.  Variables in SuperCollider come in two types.  The first looks like this:

```
(
var myVariable = 5;
myVariable.postln;
)
```

Variables are not strongly typed in SuperCollider, so there's no need to specify a type.  Within the parentheses, you can do all the normal things you'd expect from a variable.  This 'var' kind of variable is more restrictive than it appears, however.  For example, executing these two lines one after another throws an error:

```
var myVariable = 5;
myVariable.postln;
```

Variables declared with 'var' must be anchored to some larger block of code, or else they don't do anything.  Just executing var myVariable = 5; doesn't actually save anything anywhere – and after a code block ends, the 'var' variables inside it disappear and can't be used later.  Additionally, 'var' variables must be declared in the first statement of the code block or function (they can be assigned to anywhere without fear).  Anything else, such as the following, will throw an error:

```
(
'hello'.postln;
var myVariable = 5;
myVariable.postln;
)
```

You might be thinking at this point that these variables seem awfully useless, which is why SuperCollider gives us a more flexible option:

```
a = 6;
a.postln;
```

Variables with single character names in SuperCollider will behave exactly as you would expect an ordinary global variable to behave in any other programming language.  They are given to you for free, so they don't need to be declared, and they can be assigned to and used anywhere in your code (the letter 's' is reserved, so don't use that one!).  If you (understandably) don't want to use single-letter variable names, there's one more option; any name with a ~ in front of it will function exactly like a single-letter variable:

```
~myVariable = 5+6*7;
~myVariable.postln;
```

This is as good a time as any to point out that while math in SuperCollider can be done in exactly the same way as in a standard programming language, there is no operator precedence, so equations will always evaluate from left to right unless you use parentheses.

## Functions

Functions in SuperCollider are reusable pieces of code, as in most languages.  They're created using curly braces, like so:
```
~myFunction = { 'Function evaluated!'.postln; };
```
A function can contain any number of lines of code, and once we've stored it, we can evaluate it by referring to its location:
```
~myFunction.value;
```
We need to call .value on the function because functions are objects.  If you just execute ~myFunction; on its own, SuperCollider will respond back, "yep, that's a function!".  The 'value' message tells SuperCollider to actually evaluate the function.  Typically we would like to evaluate a function as soon as it's ready to go, so a fairly common sight in SuperCollider code is a structure that looks like the following:
```
(
~myFunction = {
    // Do some stuff
};

~myFunction.value;
)
```
If we hit cmd + return anywhere inside the parenthesis, we'll reassign the function to ~myFunction, thus updating it with any changes we've made, and then evaluate it immediately.  A slick alternative to this, if you don't need the function later, is to just evaluate it without assigning it, like so:
```
{ a = 5+6*7; a.postln; }.value;
```
Functions can also accept arguments.  There are two equivalent ways to write this:
```
(
~myFunction = { arg a, b; (a+b).postln; };
~myFunction.value(5, 6);
)

(
~myFunction = { |a, b| (a+b).postln; };
~myFunction.value(5, 6);
)
```
Arguments can have default values that will be overwritten only if that argument is provided.  When you take advantage of this (or perhaps always, just to be safe), be sure to refer to the arguments by name when you call the function:
```
// This will evaluate to 13
(
~myFunction = { |a = 8, b| (a+b).postln; };
~myFunction.value(b: 5);
)

// This will throw an error, because 5 overwrites a = 8, and b gets no value
(
~myFunction = { arg a = 8, b; (a+b).postln; };
~myFunction.value(5);
)

// This will evaluate to 7
(
~myFunction = { |a = 8, b| (a+b).postln; };
~myFunction.value(a: 1, b: 6);
)
```
# Generating Sound

### SinOsc and UGens

At last, time to make some noise!  Type and execute the following lines:
```
(
s.boot;
{ SinOsc.ar(freq: 440, mul: 0.5); }.play;
)
```
You should hear a constant tone coming from the left speaker.  To be more precise, it's a sine wave with frequency 440 and amplitude 0.5, as we've specified by our arguments to SinOsc.ar.  SinOsc.ar is called a Unit Generator, or UGen.  A UGen is just an object that processes or generates sound, and SuperCollider has tons of different kinds.  SinOsc is the simplest one, and the .ar signifies that the UGen runs at audio rate.  We can also write UGen.kr to get a UGen that runs at control rate.  All UGens need to send either the ar or kr message to actually process/generate sound.  

Just like how we provide arguments to our function ~myFunction.value(args), we provide named arguments to the SinOsc.ar that specify its behavior (in general, SuperCollider will helpfully tell you the arguments that a function accepts if you begin typing some in the parentheses).  The UGen created with the specified arguments is wrapped in a function using the familiar curly brace notation, and the .play message is sent to that function.  The .value message won't do anything for us here besides identify the SinOsc as a UGen – we need .play in order to actually interact with SuperCollider's audio server.  The play message isn't rigidly defined, depending on what you're using it on, but it generally means about what you'd expect: start making noise!

You'll need the s.boot; line beforehand to start SuperCollider's audio server.  You may have noticed already that when you open up SuperCollider, there are two applications running.  One is the IDE you're writing code in, and the other is a powerful audio processing server that receives your commands and is responsible for actually making the noises!  The distinction between the client and the server will become important in due time.

To stop the sound, hit cmd + period.

### Examining Your Output

Neither the .value nor .postln messages are very useful for viewing our output here – naturally you can listen to the sound created when you send .play, but if we're looking for more information about what our code is doing, where can we look?

One useful feature of SuperCollider is the ability to plot the sound waves you're creating (or anything else, really).  Just replace .play with .plot and a graph will show up that looks something like this:

![1](https://github.com/mlevatich/Introduction-To-SuperCollider/blob/master/Images/1.png)

It's a sine wave!  No surprises there.  On the other hand, if you want to look at what your sound is doing in real time, few things are easier or more illustrative than a FreqScope.  Just type and execute:
```
FreqScope.new;
```
and then play your SinOsc.  Your FreqScope will react to the sound being played and display the frequencies and amplitudes in real time.  It should look like the below.  Pretty neat!

![2](https://github.com/mlevatich/Introduction-To-SuperCollider/blob/master/Images/2.png)

### The Help Docs

The number and variety of functions and messages that are pre-packaged with vanilla SuperCollider is staggering, especially when it comes to UGens and audio effects.  You'll probably be experimenting with many of them, and the first place to look when trying to figure out how they work is SuperCollider's help documentation in the top left corner of the IDE.  You can search for classes and messages and get detailed descriptions back, with code examples to supplement them.  And to perform a quick search, you can click on any word in your editor and hit cmd+D to search for it in the documentation.

As an example, let's search for something we're going to want to know how to use: arrays.  Type Array into the editor, click on it, and hit cmd+D.  We immediately see a description that matches up with our standard understanding of arrays, and if we scroll down we can see the various class methods of the Array class, and below that, the instance methods that we can call on existing arrays.  A particularly useful class method will be Array.fill:

![3](https://github.com/mlevatich/Introduction-To-SuperCollider/blob/master/Images/3.png)

From our understanding of functions and the description and examples provided, it's easy to understand what Array.fill does.  Feel free to mess around with it in your IDE if you want to experiment with its behavior – we'll be using it soon!

# SynthDefs

### Ok, so I have all of these tools.  How do I make an instrument?

...Is the question you should be asking yourself right about now.  Never fear!  With the knowledge we've built so far, we're not far off from making all kinds of instruments.  More precisely, we'll be defining a Synth which we store on the server and can play repeatedly.  Here's what a SynthDef looks like in the editor:
```
(
SynthDef(\mySynth, { | freq, amp |
    var source;
    source = SinOsc.ar(freq: freq, mul: amp);
    Out.ar(0,source);
}).add;

Synth(\mySynth, [freq: 440, amp: 0.2]);
)
```
If you execute this code, it should play the same sine wave from our first SinOsc example.  Let's unpack what's actually happening here.

The SynthDef takes in two arguments.  The first is a name for the Synth, which is always preceded by a backslash.  The second argument is a function which begins running when the Synth is called, as in the 8th line in this example.  When we call the Synth, we identify it by name and then supply an argument array for the SynthDef's function, as I have here (freq and amp).  In this particular case, our function assigns a SinOsc with frequency and amplitude given by the arguments to a variable called source.  That source is then passed to an Out UGen with first argument 0.  We haven't seen Out.ar before, but suffice it to say that it accepts some audio as the second argument and outputs it through the "bus" specified by the first argument.  The 0th bus happens to be the left speaker, which is why the sound plays from the left speaker.  Finally, we call "add" on the entire SynthDef.

There are still questions yet to answer about this SynthDef.  What is a bus?  What is add doing?  Where did the familiar .play message go and why are we using Out instead?  Why are we accessing the Synth by its name instead of assigning it to a variable?  We're going to put these questions on hold until the tail end of this guide and focus on what we can do from a sound generation perspective with this SynthDef, based on what we know so far.

First of all, let's get our audio going to both speakers, not just the left one.  Change the first argument in Out.ar from 0 to [0, 1].  The result is a much less annoying even distribution between both speakers.  We've added another bus, 1, for Out to take our source audio to, and bus 1 happens to be the right speaker.

How can we make our sound more interesting than a simple sine wave?  A good place to start is with overtones – physical instruments create additional "overtones" whenever any one note is played, so if we want our synth to sound more "real", we can give it some overtones.

First, we add a variable 'freqs' to our SynthDef, which will be an array containing the frequencies of both the played tone and all of its overtones:
```
SynthDef(\mySynth, { | freq, amp |
    var source, freqs;
```
The most effective way to fill our freqs array is undoubtedly with Array.fill.  The formula for the overtone series of any given note is simply all of the integer multiples of the fundamental frequency.  So our Array.fill will look like this:
```
// 10 tones should be plenty to achieve the desired effect
freqs = Array.fill( 10, { arg i; freq * ( i + 1 ) } );
```
Since i starts from 0, the first element in the array will be the fundamental frequency, followed by its first 9 integer multiples.  Of course, the amplitudes of the overtones will not all be equal.  In fact, they decay according to a formula. Sounds like a job for another array!
```
SynthDef(\mySynth, { | freq, amp |
    var source, freqs, amps;
```
A simple formula for amplitudes is the amplitude of our fundamental tone (given by the amp argument) divided by the overtone number:
```
amps = Array.fill( 10, { arg i; amp / ( i + 1 ) } );
```
Next, we need to use our frequency and amplitude arrays to create 10 different SinOsc UGens, each representing a different overtone:
```
SynthDef(\mySynth, { | freq, amp |
    var source, freqs, amps;
    freqs = Array.fill( 10, { arg i; freq * ( i + 1 ) } );
    amps = Array.fill( 10, { arg i; amp / ( i + 1 ) } );
    source = Array.fill( 10, { arg i; SinOsc.ar( freq: freqs[i], mul: amps[i] ) } );
```
Lastly, we can't send each SinOsc through Out.ar individually – we need to combine them all into one signal.  This is easily accomplished with another UGen, Mix.ar, which serves exactly that purpose:
```
(
SynthDef(\mySynth, { | freq, amp |
    var source, freqs, amps;
    freqs = Array.fill( 10, { arg i; freq * ( i + 1 ) } );
    amps = Array.fill( 10, { arg i; amp / ( i + 1 ) } );
    source = Array.fill( 10, { arg i; SinOsc.ar( freq: freqs[i], mul: amps[i] ) } );
    Out.ar( [0, 1], Mix.ar(source) );
}).add;

Synth(\mySynth, [freq: 440, amp: 0.2]);
)
```
Unfortunately, while our overtones do give the sound a certain fullness that it was lacking before, it's also far more aggressive and grating.  Why don't we tone that down with a lowpass filter!
```
(
SynthDef(\mySynth, { | freq, amp |
    var source, freqs, amps;
    freqs = Array.fill( 10, { arg i; freq * ( i + 1 ) } );
    amps = Array.fill( 10, { arg i; amp / ( i + 1 ) } );
    source = Array.fill( 10, { arg i; SinOsc.ar( freq: freqs[i], mul: amps[i] ) } );
    Out.ar( [0, 1], LPF.ar(Mix.ar(source), mul: amp) ); // LPF Here!
}).add;

Synth(\mySynth, [freq: 660, amp: 0.2]);
)
```
That's a little better!  We still aren't creating anything magical, but it's easy to see how by compounding UGens on top of each other we can create a more and more complicated sound for our instrument.  Speaking of which, we haven't really been using our Synth like an instrument – we've just been playing it once immediately and letting it drone on until we stop it.  How can we fix that?

### Envelopes and the Synth as an Instrument

Ideally we would like our Synth to play a note and then stop, instead of going on and on forever.  The best tool for this job is something called an Envelope.  Envelopes are created in the following way:
```
e = Env.new( [0, 1, 0], [0.02, 1] );
```
An envelope can take several arguments, but the two most important ones are the first two: an array of levels and an array of times.  The envelope essentially represents a value which changes over time.  It begins at the first value of the levels array (zero, in this case), and climbs linearly to the second value over a period of time equal to the corresponding value in the times array.  And so on and so forth.  So this envelope starts at 0, climbs to 1 in 0.02 seconds, and then decays back to 0 over 1 second.  This is represented most clearly in a graph – if we plot the envelope, it looks like this:

![4](https://github.com/mlevatich/Introduction-To-SuperCollider/blob/master/Images/4.png)

We can turn our droning synth into a single note by treating the values the envelope takes on as multipliers for the synth's amplitude.  So we begin at volume 0, and then climb to full volume in a very short time (representative of the note's "attack") and then decay over one second as the note fades out.

In order to get our envelope to traverse it's levels in real time as we run the synth, we first need to feed it through a specialized UGen called EnvGen.  So the code which actually generates our amplitude looks like this:
```
EnvGen.ar( Env.new( [0, 1, 0], [0.02, 1] ) );
```
In our SynthDef, we can multiply each of our overtone SinOsc objects by this EnvGen.  We can think of both the SinOsc and the EnvGen as constantly generating values in real time on the server which match the graphs displayed by .plot, and in that context multiplying them together makes sense – whatever is output by each SinOsc is modified by a factor according to where we are in our envelope.  The new and improved SynthDef, then, looks like this:
```
(
SynthDef(\mySynth, { | freq, amp |
    var source, freqs, amps;
    freqs = Array.fill( 10, { arg i; freq * ( i + 1 ) } );
    amps = Array.fill( 10, { arg i; amp / ( i + 1 ) } );
    source = Array.fill( 10, { arg i; SinOsc.ar( freq: freqs[i], mul: amps[i] ) * EnvGen.ar( Env.new([0,1,0], [0.02, 1]) ) } );
    Out.ar( [0, 1], LPF.ar(Mix.ar(source), mul: amp) );
}).add;

Synth(\mySynth, [freq: 660, amp: 0.2]);
)
```
Playing this will give us something closer to a discrete event in time – a note, as though we had smacked a key on the piano.  Try messing around with the values in the envelope array and see what kind of wacky things you can get the note to do!

We aren't quite out of the woods yet though.  Synths are stored on the server, and they continue to run until they are freed by cmd+period.  That means even if one of our notes is over, and its volume has dropped to 0, it still exists on the server and is still eating up space.  You can actually witness this if you spam cmd+return on the synth and look in the bottom right corner of the IDE: the server % will go up and up until it eventually crashes.

We'd much rather free our synth automatically as soon as its volume drops to zero, i.e. as soon as the envelope ends and the note is over.  That way we'll always have room for more notes.  This is surprisingly easy to do – an additional argument to EnvGen called doneAction gives us the option to trigger an action when the envelope ends.  In order to free the synth, the value we give to that argument is Done.freeSelf.  So our final code is this:
```
(
SynthDef(\mySynth, { | freq, amp |
    var source, freqs, amps;
    freqs = Array.fill( 10, { arg i; freq * ( i + 1 ) } );
    amps = Array.fill( 10, { arg i; amp / ( i + 1 ) } );
    source = Array.fill( 10, { arg i; SinOsc.ar( freq: freqs[i], mul: amps[i] ) * EnvGen.ar(Env.new([0,1,0], [0.02, 1]), doneAction: Done.freeSelf) } );
    Out.ar( [0, 1], LPF.ar(Mix.ar(source), mul: amp) );
}).add;

Synth(\mySynth, [freq: 660, amp: 0.2]);
)
```
Now we know how to use Synths as 'instruments', to which we can give different arguments in order to play different 'notes' on the instrument!  The many UGens of SuperCollider give us virtually infinite degrees of freedom in designing an instrument which plays notes that sound exactly how we want them.  If you're looking for UGens with which you can create some interesting instruments, try checking out the following in the help docs:

WhiteNoise, PinkNoise, BrownNoise, LFPulse, Gverb, CombC, LPF

The natural next step is to ask, now that I have my instrument, how can I write code that plays my instrument in an intelligent and interesting way?  Essentially, how can I write code that makes music?

# Algorithmic Composition

### Patterns

There are many, many tools you can use for composition in SuperCollider, but since I can't go over them all here, I'll just be focusing on patterns, and the wide range of options they provide you for algorithmic composition.

A pattern, at its most basic, is really nothing more than a sequence of values.  There are many different pattern classes that all do different things, but consider a simple example, Pseq():
```
p = Pseq([1, 2, 3, 4], 6);
```
The first argument to Pseq is the list of values to be iterated over, and the second argument is how many times to iterate.  So the values generated by this pattern would be 1, 2, 3, 4, 1, 2, 3, 4, 1, 2...etc.  We can see this by converting the pattern to a stream and then calling next on it:
```
p = Pseq([1, 2, 3, 4], 6);
p.next; // 1
p.next; // 2
```
Another simple pattern class is Prand:
```
p = Prand(#[1, 3, 5, 7], inf);
```
Prand generates a pattern by picking a random element from the list, a number of times specified by the second argument.  In this case, that argument is inf, meaning the pattern will go on forever.

Patterns can be nested inside of each other, and nested patterns are embedded in the outer pattern's output.  For example, this pattern
```
p = Prand([
    Pseq([1,2,3], 2),
    Pseq([4,5,6], 2),
    Pseq([7,8,9], 2)], 
10);
```
will randomly pick one of the three sequences, loop through it twice, and then randomly pick again, until it has made 10 random selections for a total of 60 values in the pattern.

By using these patterns as arguments to a synthDef, we can play notes of certain durations and frequencies and amplitudes in any order we want, creating music!  But how exactly do we do that?

### Pbind and Ppar

Pbind is another pattern class, and it serves as the intersection between patterns and synths.  Pbind accepts as arguments the Synth to be used as an instrument, the arguments to that synth, and the duration of each value in the pattern. If we use our synth from above, we might write a Pbind like so:
```
(
p = Pbind(
    \instrument, \mySynth,
    \freq, Prand(#[440, 660, 220, 880], inf),
    \amp, 0.2,
    \dur, 0.25
)
p.play;
)
```
The \instrument argument specifies the synth used, while the \dur argument gives the duration of each value.  The other arguments must match up with the argument names in the synthDef.  The values of those arguments are then fed as arguments into the synth.  So in this example, in intervals of 0.25, \mySynth would be called with a random frequency argument matching one of the four values in the pattern.  It's a rather pretty effect.

We can either use Prand to make music in real time, algorithmically, or Pseq to hardcode a composition with specific note values and note durations (or a combination of the two).  Either way, if the synth is our instrument, then the Pbind and its associated patterns act as a kind of musical score, giving the dynamics, note values, pitches, and any other qualities with which the instrument is to be played.

If we want multiple instruments playing at once, we use Ppar, which is a pattern class that accepts a list of patterns as arguments, and when played, starts all of the patterns simultaneously.  The most common use of Ppar is to provide it a list of Pbinds which all correspond to different instruments, which are then played together to create music!  If you need different instruments to start at different points in time, Ptpar allows you to give each pattern in the list an anacrusis of some duration by providing an additional argument.

By way of example, [here's](https://pastebin.com/Tn0nGEH1) some code for a little chiptune song I wrote using synthdefs, Ptpar, Pbind, and Pseq!

### TempoClock

I mentioned that the \dur argument specifies an interval for which the synth is to be played.  To be more specific, the interval can be thought of as a beat, where 0.25 is a quarter note, 0.5 is a half note, etc.  The overall tempo can be controlled using a global object called a TempoClock.  The easiest way to set it is the following:

TempoClock.default.tempo = 3;

The value which this variable is set to represents the beats per second. 

# The Server and Synths as Effects

### Why are synthDefs so weird?

Back when we were talking about synthDefs I said that I would come back to some of the stranger things about them, like the use of Out.ar, audio buses, and the add message.  All of these oddities basically stem from the fact that synths live on the SuperCollider audio server.  Nothing in this guide so far has hinged upon that distinction, but it's an important thing to understand if one wishes to dive deep into the language.

On the server, a synth is a type of "node".  The server is comprised of these nodes, which can be organized into "groups".  Calling .add on the synthDef then can be thought of as sending the synthDef to the server and assigning it to a node.

Out.ar() is just handling explicitly what convenience functions like .play handle implicitly – sending audio through an audio bus.  In order to really understand this, though, we have to talk about bussing.  Buses are responsible for sending signals from one place to another.  Signals can be sent from a synth to the audio hardware (using 0 and 1 for the left and right speakers, as we've been doing; audio hardware always gets the lowest bus numbers by default), but they can also be sent from synth to synth.  This brings us to our final topic: synths as effects.

### Synths as Effects

Since we can use bussing to send the output of synths not only to the audio hardware but also to other synths, we can generate an audio signal using a synth and then send it to a different synth to further transform it.  Why would we want to do this?  Well, we don't always have to think of a synth as an instrument.  A synth could instead be an effect that applies a transformation to an input signal coming from some bus, and then sends it to the audio hardware.  A synth could apply reverb to an incoming signal, as a simple example:
```
~bus = Bus.audio(s, 2);

SynthDef(\some_random_synth, {| args |
        // Code to generate some signal
        Out.ar(~bus, signal);
    }).add;

SynthDef(\verb, {
        var tones;
        tones = In.ar(~bus, 2); // Here's where we get our signal from!
        Out.ar([0,1], GVerb.ar(tones,drylevel:dry));
}).add;
```
Here we create a bus, and our synth sends a signal through that bus, whereupon the reverb synth receives it and applies the transformation.

Using synths as effects offers you more modularity than combining all of your effects into a single instrument – if you create a really cool effect, you can apply it to multiple different instruments in different ways if you keep it in its own synth!

# Bye!

That just about covers the basics of SuperCollider!  Hopefully this guide helped you get on your feet and get started working in this powerful language.  With so many different capabilities to explore, learning SuperCollider is more about exploration than anything else – I only hope this introduction helps to kickstart that process.
