+++
date = "2016-10-19T20:54:22+05:30"
sticky = false
tags = ["regex"]
title = "Regular Expression Lookaround"

+++

Regular expression *lookahead* and *lookbehind*, collectively called *lookaround*, are zero-length assertions. They're called zero-length assertions because they do not consume characters in the string, but only assert whether a match is possible or not.

Lookarounds are very useful, because they allow you to create regular expressions that are impossible to create without them, or that would get very unmanageable without them.

#### Negative lookahead

Negative lookahead is useful when you want to match only those instances of a pattern that are not followed by another pattern. For example, the regular expression `a(?![bcd])` matches the character `a` when it isn't followed by any of the characters `b`, `c` and `d`.

Let's take a look at a Python snippet demonstrating a practical use-case for negative lookahead:

{{< highlight python >}}
import re

# Match all commas not followed by a space
pattern = re.compile(',(?!\s)')

text = '''
You will know (the good from the bad) when you are calm, at peace. Passive.
A Jedi uses the Force for knowledge and defense,never for attack.
'''

pattern.findall(text)  # [',']
{{< /highlight >}}

#### Negative lookbehind

Negative lookbehind is useful when you want to match only those instances of a pattern that are not preceded by another pattern. For example, the regular expression `(?<![bcd])a` matches the character `a` only when it isn't preceded by any of the characters `b`, `c` and `d`.

Let's take a look at a Python snippet demonstrating a practical use-case for negative lookbehind:

{{< highlight python >}}
import re

# Match all instances of the pattern `Vader` not preceded by the pattern `Darth\s`
pattern = re.compile('(?<!Darth\s)Vader')

text = '''
If you end your training now — if you choose the quick and easy path as
Vader did — you will become an agent of evil.
'''

pattern.findall(text)  # ['Vader']
{{< /highlight >}}

#### Positive lookahead

Positive lookahead is useful when you want to match only those instances of a pattern that are followed by another pattern. For example, the regular expression `a(?=[bcd])` matches the character `a` only when it is followed by any of the characters `b`, `c` and `d`.

Let's take a look at a Python snippet demonstrating a practical use-case for positive lookahead:

{{< highlight python >}}
import re

# Match all instances of the pattern `dark` that are followed by the pattern `\sside`
pattern = re.compile('dark(?=\sside)')

text = '''
Yes, a Jedi’s strength flows from the Force. But beware of the dark side.
Anger, fear, aggression; the dark side of the Force are they. Easily they
flow, quick to join you in a fight. If once you start down the dark path,
forever will it dominate your destiny, consume you it will, as it did
Obi-Wan’s apprentice.
'''

pattern.findall(text)  # ['dark', 'dark']
{{< /highlight >}}

#### Positive lookbehind

Positive lookbehind is useful when you want to match only those instances of a pattern that are preceded by another pattern. For example, the regular expression `(?<=[bcd])a` matches the character `a` only when it is preceded by any of the characters `b`, `c` and `d`.

Let's take a look at a Python snippet demonstrating a practical use-case for negative lookahead:

{{< highlight python >}}
import re

# Match all instances of the pattern `I` that are preceded by the pattern `have\s`
pattern = re.compile('(?<=have\s)I')

text = '''
Ready are you? What know you of ready? For eight hundred years have I trained
Jedi. My own counsel will I keep on who is to be trained. A Jedi must have the
deepest commitment, the most serious mind. This one a long time have I watched.
All his life has he looked away… to the future, to the horizon. Never his mind
on where he was. Hmm? What he was doing. Hmph. Adventure. Heh. Excitement. Heh.
A Jedi craves not these things. You are reckless.
'''

pattern.findall(text)  # ['I', 'I']
{{< /highlight >}}
