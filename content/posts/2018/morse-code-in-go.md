+++
author = "James D Hughes"
categories = ["golang"]
date = 2018-08-15T02:52:19Z
description = ""
draft = false
slug = "morse-code-in-go"
tags = ["golang"]
title = "Learning Go - Morse Code!"

+++


I was watching Churchill's secret agents on Netflix the other day

When all of a sudden I got a hit of nostalgia when they got to the portion on morse code.  Such an ingenious little method of communcation that I still, to this day, couldn't follow for the life of me!
I took a class on encryption and one of the first concepts they spoke of was morse code.  Taking language and transcribing it into a not-so-easy to decipher combination of beeps and blips to send messages over a wire.  
Some other fun facts for those of you that haven't taken any introductory encryption classes, morse code is designed very closely following a common english language frequency table.  That is, a table that determines the frequency at which a letter appears in the alphabet.  Letters that appear frequently are, in morse code, encoded using fewer and shorter . and -'s!
I've added in a column to the table found at:
http://pi.math.cornell.edu/~mec/2003-2004/cryptography/subs/frequencies.html
and put in the morse code equilvalent for a quick reference

<table>
  <thead>
      <th>Letter</th>
      <th>Frequency %</th>
      <th>Morse</th>
  </thead>
  <tbody>
      <tr>
          <td>E</td>
          <td>12.02</td>
          <td>.</td>
      </tr>
      <tr>
          <td>T</td>
          <td>9.10</td>
          <td>-</td>
      </tr>
      <tr>
          <td>A</td>
          <td>9.10</td>
          <td>.-</td>
      </tr>
      <tr>
          <td>O</td>
          <td>7.68</td>
          <td>---</td>
      </tr>
      <tr>
          <td>I</td>
          <td>7.31</td>
          <td>..</td>
      </tr>
      <tr>
          <td colspan = "3" style="font-style:italic;text-align:center">(Several Letters Later...)</td>
      </tr>
      <tr>
          <td>Z</td>
          <td>0.07</td>
          <td>--..</td>
      </tr>
  </tbody>
</table>

I've spent a load of time trying to learn Go and, since this would be a simple little task, I've decided to use Go to put it together!

So, what we're going to make is a system that either:
1. Takes in a plaint-text string and converts it into a series of .'s and -'s 
2. Takes in a bunch of .'s and -'s and transcribes them into plain text

Making the directory:  %GOPATH%/src/{your_github_slug}/go-morse and adding in a file called 'main.go'

```go
package main

func main() {
    fmt.Println("Hello, Morse Code!");
}
```
You can make sure everything is working by running `go run main.go` from your command line - but this step is probably skippable.

To make this work, we're going to require a list of all the characters in morse code. A quick google seach should yeild this for you.  Here's what I did to make it go. I separated it out into another file called `codes.go`

```go
package main

var morsemap = map[string]string{
	"A": ".-",
	"B": "-...",
	"C": "-.-.",
	"D": "-..",
	"E": ".",
	"F": "..-.",
	"G": "--.",
	"H": "....",
	"I": "..",
	"J": ".---",
	"K": "-.-",
	"L": ".-..",
	"M": "--",
	"N": "-.",
	"O": "---",
	"P": ".--.",
	"Q": "--.-",
	"R": ".-.",
	"S": "...",
	"T": "-",
	"U": "..-",
	"V": "...-",
	"W": ".--",
	"X": "-..-",
	"Y": "-.--",
	"Z": "--..",
	"1": ".----",
	"2": "..---",
	"3": "...--",
	"4": "....-",
	"5": ".....",
	"6": "-....",
	"7": "--...",
	"8": "---..",
	"9": "----.",
	"0": "-----",
	".": ".-.-.-",
	",": "--..--",
	"?": "..--..",
	"!": "-.-.--",
	"-": "-....-",
	"/": "-..-.",
	"@": ".--.-.",
	"(": "-.--.",
	")": "-.--.-",
}

var characterMap = map[string]string{}

func init() {
	for key, value := range morsemap {
		characterMap[value] = key
	}
}
```
The morsemap is an encoding chart for changing characters into their morse equivalent.  The `init()` function then takes the first defined list and creates an inverse list. This will help when looking up morse characters to actual letters.

Now we'll need an encoder / decoder.  I'm going to put those into a file called `encoder.go`

```go
package main

import "strings"

func encodeMessage(message, letterSplitter string) string {
	var output string
	message = strings.ToUpper(message)
	words := strings.Split(message, " ")
	for _, word := range words {
		word := encodeWord(word, letterSplitter)
		if word != "" {
			output += word
		}
	}
	return output
}

func encodeWord(word, letterSplitter string) string {
	var morse string
	for i := 0; i < len(word); i++ {
		code := morsemap[word[i:i+1]]
		if code != "" {
			morse += code + letterSplitter
		}
	}
	return morse
}
```

This is pretty straight forward.  The encodeMessage function takes a message and a delimiter as a string. It then converts the message to upper case (for ease of use) and then splits the words into manageamble chunks. It iterates through each word and then encodes the word and appends it to the output string.  The delimiter is used as a 'stop' character. This could just be a space or you could put in a `\` or `/` or something to make it easier to read. We'll make that configurable using flags!

We can test this now - in your main function let's make magic happen!
```go
func main(){
  message := "Hello, World!"
  delimiter := " "
  fmt.Println(encodeMessage(message, delimiter))
}
```
You'll have to build the project now, as we've split our code into multiple files. Do this by calling `go build`  This will generate a new executable file which you can run and you should get a message like this:

```
.\morse-go.exe
.... . .-.. .-.. --- --..-- .-- --- .-. .-.. -.. -.-.--
```

Feel free to confirm that against the list :)

So we can generate morse code. What happens when we receive a message back? Well we better implement the decoder now!

I'm just going to put the code in the `encoder.go` file for ease of use.

```go
...
func decodeMessage(message, letterSplitter string) string {
  var decodedMessage string
  var words := strings.Split(message, letterSplitter)
  for _, letter := range letters {
    decodedMessage += decodeLetter(strings.Trim(letter, " ")) + " "
  }
  return decodedMessage
}

func decodeLetter(code string) string {
  return characterMap[code]
}
```

To finish the circle, let's add the output of our previous example into our test.
```go

func main() {
  message := "Hello, World!"
  morse := ".... . .-.. .-.. --- --..-- .-- --- .-. .-.. -.. -.-.--"
  delimiter := " "
  fmt.Println(encodeMessage(message, delimiter))
  fmt.Println(decodeMessage(morse, delimiter))
}
```
Your output will look like so
```go
.\go-morse
.... . .-.. .-.. --- --..-- .-- --- .-. .-.. -.. -.-.--
H E L L O , W O R L D !
```

This isn't all that useful though.  It'd be nice if we could pass some arguments to the program so that we could encode and decode messages on the fly. We can do this by using golangs built in `flags` library.

You can set up a flag by adding the following code to your `main` function
```go
func main() {
  messagePtr := flag.String("message", "", "The message to encode")
}
```
This will assign a pointer to the CLI flag called "message" and will default it to an empty string if it is not set.  You can do these for the remainder of our variables and your main will now look something like this:

```go
func main() {
	messagePtr := flag.String("message", "", "the message to encode")
	morsePtr := flag.String("morse", "", "message in morse code to be decoded")
	letterSplitter := flag.String("delimiter", " ", "the delimiter for morse characters")
	flag.Parse()
	if *messagePtr == "" && *morsePtr == "" {
		log.Fatal(errors.New("No message to encode or decode. Provide one of -message or -morse"))
	}
	if *letterSplitter == "-" || *letterSplitter == "." {
		log.Fatal(errors.New("Letter Delimiter cannot be a \"-\" or \".\""))
	}
	if *messagePtr != "" {
		fmt.Println(encodeMessage(*messagePtr, *letterSplitter))
	}
	if *morsePtr != "" {
		fmt.Println(decodeMessage(*morsePtr, *letterSplitter))
	}
}
```

There you have it! Compile your code using `go build` and then run `./morse-go -message "This is super neat!"` and you'll get a response of:
```
$ .\morse-go.exe -message "This is super neat!" -delimiter="\"
$ - .... .. ... .. ... ... ..- .--. . .-. -. . .- - -.-.--
```
To check our work, run the inverse:
```
.\morse-go.exe -morse "- .... .. ... .. ... ... ..- .--. . .-. -. . .- - -.-.--"
T H I S I S S U P E R N E A T !
```
and there you have it! A super simple morse code utility.

To think that people say watching TV doesn't amount to anything productive :-)

The code for this is live at my [Github](https://github.com/Daimonos/morse-go)

