---
layout: inner
position: center
title: Java Langauge Feature Abuses
date: 2021-11-01 14:15:00
categories: Java
tags: Java langauge-abuse
lead_text: "If you can successfully compile and run, your program is syntactically correct. But if it is semantically wrong, you may be abusing the language constructs. Below I am trying to explain two of such language abuses, that I read somewhere coincidently."
project_link: 'http://steephengeorge.github.io/java/2021/11/01/Java-Langauge-Feature-Abuses.html'
---
# Java Langauge Feature Abuses


If you can successfully compile and run, your program is syntactically
correct. But if it is semantically wrong, you may be abusing the
language constructs. Below I am trying to explain two of such language
abuses, that I read somewhere coincidently.

## Misuse of concurrency constructs

I have to add integers from a list to an already known value. I can
tackle it in several ways.

{% highlight java %}

//By using a range loop

public static Integer adder(List\<Integer\> numbers) {

	Integer sum = 10;

	for(Integer number: numbers){

	sum += number;

	}

	return sum;

}

//By using lambda with reduce function

public static Integer adder(List\<Integer\> numbers) {

	Integer sum = 10;

	sum = numbers.stream().reduce(sum, Integer::*sum*);

	return sum;

}

{% endhighlight %}

These are all good. But see the next version.

{% highlight java %}

public static Integer adder(List\<Integer\> numbers) {

	Integer sum = 10;

	numbers.stream().forEach(number -\>

		sum += number );

	return sum;

}

{% endhighlight %}

I will get the following error:

{% highlight bash %}

java: local variables referenced from a lambda expression must be final
or effectively final

{% endhighlight %}

I can solve this error by abusing the language as follows.

{% highlight Java %}

public static Integer adder(List\<Integer\> numbers) {

	AtomicInteger sum = new AtomicInteger(10);

	numbers.stream().forEach(number -\>

		sum.addAndGet(number));

	return sum.get();

}

{% endhighlight %}

Someone who reviews this code should ask at least two questions:

-   Is concurrency in play in this context?

-   Are we using the appropriate language construct to tackle the problem?

If there is no concurrent scenario in the context, AtomicInteger is
overkill as well as language abuse. Did you see this kind of code
somewhere?

## AtomicBoolean is not your toy!!

AtomicBoolean is different from Boolean only in one regard the
operations we can do on AtomicBoolean is atomic at the hardware level.
Atomic operations can generate results by executing a single hardware
instruction.

{% highlight Java %}

public boolean updateCurrentStatus(boolean leftStatus,

boolean rightStatus){

	AtomicBoolean currentStatus = new AtomicBoolean(true);

	currentStatus.set(leftStatus \|\| rightStatus);

	return currentStatus.get();

}

{% endhighlight %}

The above code snippet successfully compiles and gives you a result, but
it is not semantically right.

The normal boolean operations OR or AND are not atomic. In the above
code snippet, we packed the normal boolean operations within the
parameter of set operation for AtomicBoolean. So we are mixing a
non-atomic operation inappropriately with an atomic variable. It is a
code smell. My gut feeling is, you don\'t need AtomicBoolean in this
context. If you argue on thread safety, simpy this snippet is neither
thread-safe nor atomic.
