# CS550 Fall Term Work

## Ventre Program
> Using core concepts from the term to create a class-based representation of the loveable Choate Jazz Band director Phil Ventre, who gives iconic quotes based on his mood. Allows interaction with "Ventre" through a typing game of variable difficulty that can change his mood and then show the user how they did by playing a song (Mueva Los Huesos, Machito, or Told You So) adjusted to reflect how well they did based on a tuning and rythym factor. 

#### Ventre Program Code:
```python
import time
import string
import random
import os
import _thread
import time
import sys
import pysynth
from play_wav import Sound
import copy
from random import shuffle


noteswo = ["c3", "c#3", "d3", "d#3", "e3", "f3", "f#3", "g3", "g#3", "a3", "a#3", "b3",
          "c", "c#", "d", "d#", "e", "f", "f#", "g", "g#", "a", "a#", "b",
          "c5", "c#5", "d5", "d#5", "e5", "f5", "f#5", "g5", "g#5", "a5", "a#5", "b5",
          "c6", "c#6", "d6", "d#6", "e6", "f6", "f#6", "g6", "g#6", "a6", "a#6", "b6"]

def changepitch(song, perc):
    #Changes the notes on the song based on the percentage

    if perc >= 1:
        return song
    song2 = song
    songlen = len(song)
    noteschange = int(songlen // (1/(1 - perc)))
    #Changes the number of notes changed
    for i in range(noteschange):
        note = random.randint(0, songlen-1)
        (a,b) = song2[note]
        c = 0
        if a == 'r':
            a = noteswo[random.randint(12,23)]
            continue
        if a[-1] == '*':
            c = noteswo.index(a[:-1])
        else:
            c = noteswo.index(a)
        #Changes how much the note is changed
        a = noteswo[c+int(random.randint(-6, 6)//(1/(1 - perc)))]
        song2[note] = (a,b)
    return song2


def changerhythm(song, perc):
    #Changes the rhythm of the song based on the percentage
    if perc >= 1:
        return song
    song2 = song
    songlen = len(song)
    noteschange = int(songlen // (1/(1 - perc)))
    for i in range(noteschange):
        note = random.randint(0, songlen-1)
        (a,b) = song2[note]
        b = random.randint(4, 10)
        #makes it a value between 4 and 10
        song2[note] = (a,b)
    return song2

def interrupt(difficulty):
    if difficulty == 1:
        delay = 10
    elif difficulty == 3:
        delay = 15
    elif difficulty == 5:
        delay = 21
    time.sleep(delay)
    print("\nTime's Up! (Press Enter)")
    #Adjusts the delay before interrupting based on the difficulty/associated string length.
    #Took inspiration from Stack Overflow.
    #Used later in a sub-string to interrupt user input.

def clamp(low, n, hi):
    return sorted([low, n, hi])[1]
#Took inspiration from Stack Overflow.
#Cute function that "clamps" the number n within designated range (low-hi).

def correct(playnotes, play, difficulty):
    notes = 0
    for n in range (0, clamp(0, len(play), len(playnotes))):
    #Clamps peak range so that the length of the inputted string can be less than the actual string,
    #but cannot exceed it (thereby accessing an index outside of the actual string).
        if playnotes[n] == play[n] and playnotes[n] != ' ' and play[n] != ' ':
            notes += 1
    #Evaluates and returns the number of correct "notes" in the input (non-space characters).
    return clamp(0, notes, len(playnotes)-playnotes.count(' '))

def rythym(playnotes, play, difficulty):
    rythym = 0
    for n in range (0, clamp(0, len(play), len(playnotes))):
        if playnotes[n] == play[n] and play[n] == ' ':
            rythym += 1
    #Evaluates and returns the number of correct "rythym characters" in the input (spaces).
    return clamp(0, rythym, playnotes.count(' '))

def level():
    loop = True
    while loop:
        level = input('Choose Easy (4 bars), Medium (8 bars), or Hard (12 bars): ')
        if level.lower() == 'easy':
            game = 1
            loop = False
        elif level.lower() == 'medium':
            game = 3
            loop = False
        elif level.lower() == 'hard':
            game = 5
            loop = False
        else:
            input(""+str(level)+" is not a level! (Press Enter)")
            os.system('clear')
            loop = True
    #User-determined difficulty, modulates string length, time, etc.
    #Also determines length of audio file played.
    #Contains error-checking loop.
    return game

def newplay(level, notes):
    playing = []
    for d in range(0, level*4):
        playing.append(notes[random.randint(0, 47)])
        if d//4 == 0:
            playing += [" ", " ", " "]
    #Adds some spaces/rythym characters.
    shuffle(playing)
    #Shuffles the notes.
    playnotes = ''.join(playing)
    #Took inspiration from Stack Overflow. Joins an list into a string.
    return playnotes

class Ventre:
    #Constructor
    def __init__(self):
        self.name = 'Ventre'
        self.nice = random.randint(2, 6)
        self.humor = random.randint(4, 8)
        self.salty = random.randint(4, 8)
        self.mad = random.randint(2, 6)
        self.mood = 0
    #Mood variables
    def stats(self):
        print('Ventre Stats: ')
        self.mood = ((self.nice) + self.humor//2) - (2*(self.salty) + (self.mad))
    #Mood Range: -30 to 15
    #Derived from mood variables, each weighing in differently.
        if (-30 <= self.mood <= -20):
            state = 'Pretty Angry'
        elif (-20 <= self.mood <= -15):
            state = 'Kinda Angry'
        elif (-15 <= self.mood <= -10):
            state = 'Kinda Salty'
        elif (-10 <= self.mood <= 0):
            state = 'Pretty Good'
        elif (0 <= self.mood <= 15):
            state = 'Happy'

        print('Ventre Mood: ' +str(state))
        print('-------------------------')
        print('Nice: '+str(self.nice)+'/10')
        print('Humor: ' +str(self.humor)+'/10')
        print('Salty: ' +str(self.salty)+'/10')
        print('Mad: ' +str(self.mad)+'/10')
        print('-------------------------')
        print('Quote: Ask Ventre for a quote on how he feels.')
        print('Conduct: Play for Maestro Ventre to improve (or worsen) his mood!')
        print('Nothing: Or play for him another day...')
        print('-------------------------')
        #User interface explaining Ventre actions and updating mood.


    def timing(self):
        hard = level()
        #Calls function that determines difficulty.
        os.system('clear')
        s1 = Sound()
        
        print('Picking your song! (DO NOT PRESS ENTER)')

        notes = ["C", "C#", "D", "D#", "E", "F", "F#", "G", "G#", "A", "A#", "B",
         "C", "C#", "D", "D#", "E", "F", "F#", "G", "G#", "A", "A#", "B",
         "C", "C#", "D", "D#", "E", "F", "F#", "G", "G#", "A", "A#", "B",
         "C", "C#", "D", "D#", "E", "F", "F#", "G", "G#", "A", "A#", "B"]
         #Pre-defined list with all possible notes.


        lick = random.randint(1, 3)
        #Change the lick
        if lick == 1:
            bpmval = 175
            k = 12
            #12
            #Change how long the lick is based on the difficulty (4 bars for easy, 8 for medium and 12 for hard)
            if hard == 1:
                playnoteswo = [
                    (noteswo[k+9] + "*", 4),(noteswo[k+11], 8),(noteswo[k+12], 8),('r', 8),(noteswo[k+4] + "*", 4),(noteswo[k+3] + "*", 8),
                    ("r", 8), (noteswo[k+11] + "*", -4), (noteswo[k-1] + "*", 4), (noteswo[k], 8), (noteswo[k+1], 8),
                    (noteswo[k+2] + "*", 4), (noteswo[k+4], 8), (noteswo[k+5] + "*", 8), ("r", 8), (noteswo[k+7] + "*", 4), (noteswo[k+4] + "*", 8),
                    ("r", 8), (noteswo[k] + '*', 4), (noteswo[k-1] + "*", 8), (noteswo[k-3] + "*", 4)]
            elif hard == 3:
                playnoteswo = [
                    (noteswo[k+9] + "*", 4),(noteswo[k+11], 8),(noteswo[k+12], 8),('r', 8),(noteswo[k+4] + "*", 4),(noteswo[k+3] + "*", 8),
                    ("r", 8), (noteswo[k+11] + "*", -4), (noteswo[k-1] + "*", 4), (noteswo[k], 8), (noteswo[k+1], 8),
                    (noteswo[k+2] + "*", 4), (noteswo[k+4], 8), (noteswo[k+5] + "*", 8), ("r", 8), (noteswo[k+7] + "*", 4), (noteswo[k+4] + "*", 8),
                    ("r", 8), (noteswo[k] + '*', 4), (noteswo[k-1] + "*", 8), (noteswo[k-3] + "*", 4),
                    ("r", 8), (noteswo[k+4], 8), (noteswo[k+5], 8), (noteswo[k+6], 8), (noteswo[k+7], 4), (noteswo[k-2], 8),(noteswo[k-3] + "*", 8),
                    ("r", 8), (noteswo[k], 4), (noteswo[k+5], 4), (noteswo[k-1], 8),(noteswo[k], 8),(noteswo[k+2], 8),
                    (noteswo[k+4], 8), (noteswo[k+5], 16), (noteswo[k+4], 16), (noteswo[k+3], 8), (noteswo[k+4], 8), (noteswo[k+5], 8),(noteswo[k+6], 8),(noteswo[k+7], 8),(noteswo[k+8], 8),
                    (noteswo[k+9], 4), ("r", 8), (noteswo[k+9], 8), ("r", 2)
                    ]
            elif hard == 5:
                playnoteswo = [
                    (noteswo[k+9] + "*", 4),(noteswo[k+11], 8),(noteswo[k+12], 8),('r', 8),(noteswo[k+4] + "*", 4),(noteswo[k+3] + "*", 8),
                    ("r", 8), (noteswo[k+11] + "*", -4), (noteswo[k-1] + "*", 4), (noteswo[k], 8), (noteswo[k+1], 8),
                    (noteswo[k+2] + "*", 4), (noteswo[k+4], 8), (noteswo[k+5] + "*", 8), ("r", 8), (noteswo[k+7] + "*", 4), (noteswo[k+4] + "*", 8),
                    ("r", 8), (noteswo[k] + '*', 4), (noteswo[k-1] + "*", 8), (noteswo[k-3] + "*", 4),
                    ("r", 8), (noteswo[k+4], 8), (noteswo[k+5], 8), (noteswo[k+6], 8), (noteswo[k+7], 4), (noteswo[k-2], 8),(noteswo[k-3] + "*", 8),
                    ("r", 8), (noteswo[k], 4), (noteswo[k+5], 4), (noteswo[k-1], 8),(noteswo[k], 8),(noteswo[k+2], 8),
                    (noteswo[k+4], 8), (noteswo[k+5], 16), (noteswo[k+4], 16), (noteswo[k+3], 8), (noteswo[k+4], 8), (noteswo[k+5], 8),(noteswo[k+6], 8),(noteswo[k+7], 8),(noteswo[k+8], 8),
                    (noteswo[k+9], 4), ("r", 8), (noteswo[k+9], 8), ("r", 2),
                    ("r", 8), (noteswo[k+8], 8), (noteswo[k+9], 8),(noteswo[k+11], 8), (noteswo[k+12], 4), (noteswo[k+4], 8), (noteswo[k+3] + "*", 8),
                    ("r", 8), (noteswo[k+6]+ "*", 4), (noteswo[k+11]+ "*", 8), ("r", 8), (noteswo[k-1], 8), (noteswo[k], 8), (noteswo[k+1],8),
                    (noteswo[k+2], 8),(noteswo[k+1], 8),(noteswo[k+2], 8),(noteswo[k+4], 8),(noteswo[k+5], 8),(noteswo[k+7]+ "*", 8),("r", 8), (noteswo[k+4]+ "*", 8),
                    ("r", 8), (noteswo[k]+ "*", 4), (noteswo[k-1], 8), (noteswo[k-3]+ "*", 4), ("r", 4)
                    ]
        elif lick == 2:
            bpmval = 145
            k = 10
            #12
            if hard == 1:
                playnoteswo = [
                    (noteswo[k+24], -2), (noteswo[k+23], 4), (noteswo[k+22], 2), (noteswo[k+15], 2), (noteswo[k+21], 1/2),
                    (noteswo[k+21], -2), (noteswo[k+20], 4), (noteswo[k+19], 2), (noteswo[k+12], 2), (noteswo[k+18], 1/2)
                  ]
            elif hard == 3:
                playnoteswo = [
                    (noteswo[k+24], -2), (noteswo[k+23], 4), (noteswo[k+22], 2), (noteswo[k+15], 2), (noteswo[k+21], 1/2),
                    (noteswo[k+21], -2), (noteswo[k+20], 4), (noteswo[k+19], 2), (noteswo[k+12], 2), (noteswo[k+18], 1/2),
                    (noteswo[k+18], -2), (noteswo[k+17], 4), (noteswo[k+16], 2), (noteswo[k+9], 2), (noteswo[k+21], -2), (noteswo[k+20], 4), (noteswo[k+19], 2), (noteswo[k+12], 2)
                  ]
            elif hard == 5:
                playnoteswo = [
                    (noteswo[k+24], -2), (noteswo[k+23], 4), (noteswo[k+22], 2), (noteswo[k+15], 2), (noteswo[k+21], 1/2),
                    (noteswo[k+21], -2), (noteswo[k+20], 4), (noteswo[k+19], 2), (noteswo[k+12], 2), (noteswo[k+18], 1/2),
                    (noteswo[k+18], -2), (noteswo[k+17], 4), (noteswo[k+16], 2), (noteswo[k+9], 2), (noteswo[k+21], -2), (noteswo[k+20], 4), (noteswo[k+19], 2), (noteswo[k+12], 2),
                    (noteswo[k+24], -2), (noteswo[k+23], 4), (noteswo[k+24], 2), (noteswo[k+19], 2), (noteswo[k+22], 3), (noteswo[k+22], 3), (noteswo[k+22], 3), (noteswo[k+24], 3)
                  ]

            #based on how long the song is, delay the game for that amount of time so the song can play

        elif lick == 3:
            bpmval = 120
            k = 17
            #17
            #Change how long the lick is based on the difficulty (4 bars for easy, 8 for medium and 12 for hard)
            if hard == 1:
                playnoteswo = [('r', 4), (noteswo[k+4] + "*", 2),(noteswo[k+7]+ "*", 4),(noteswo[k+9], 7),(noteswo[k+3], 14),(noteswo[k+2], 7),
                    (noteswo[k], -4), (noteswo[k+12], 7), (noteswo[k+12], -4), (noteswo[k+9] + "*", 4), (noteswo[k+7], 7), 
                    (noteswo[k+9], 1), ("r", -4), ('r', 2),
                    ]
            elif hard == 3:
                playnoteswo = [('r', 4), (noteswo[k+4] + "*", 2),(noteswo[k+7]+ "*", 4),(noteswo[k+9], 7),(noteswo[k+3], 14),(noteswo[k+2], 7),
                    (noteswo[k], -4), (noteswo[k+12], 7), (noteswo[k+12], -4), (noteswo[k+9] + "*", 4), (noteswo[k+7], 7), 
                    (noteswo[k+9], 1), ("r", -4), ('r', 2), (noteswo[k+4] + "*", 2),(noteswo[k+7]+ "*", 4),(noteswo[k+9], 7),(noteswo[k+12], 14),
                    (noteswo[k+9], 7), (noteswo[k+9], -4), (noteswo[k+3], 7), (noteswo[k+3], -4), (noteswo[k] + "*", 4), 
                    (noteswo[k+2], 2), 
                    ]
            elif hard == 5:
                playnoteswo = [('r', 4), (noteswo[k+4] + "*", 2),(noteswo[k+7]+ "*", 4),(noteswo[k+9], 7),(noteswo[k+3], 14),(noteswo[k+2], 7),
                    (noteswo[k], -4), (noteswo[k+12], 7), (noteswo[k+12], -4), (noteswo[k+9] + "*", 4), (noteswo[k+7], 7), 
                    (noteswo[k+9], 1), ("r", -4), ('r', 2), (noteswo[k+4] + "*", 2),(noteswo[k+7]+ "*", 4),(noteswo[k+9], 7),(noteswo[k+12], 14),
                    (noteswo[k+9], 7), (noteswo[k+9], -4), (noteswo[k+3], 7), (noteswo[k+3], -4), (noteswo[k] + "*", 4), 
                    (noteswo[k+2], 2), 
                    ]
        #Play the song on a seperate thread behind the wait-time.
        pysynth.make_wav(playnoteswo, bpm = bpmval, pause = 0., boost = 1.1, fn = "song.wav", silent = True)
        _thread.start_new_thread(s1.playFile, ("song.wav",))

        #Based on how long the song is, delay the game for that amount of time so the song can play.
        if hard == 1:
            delay = random.randint((16*60)//175 + 2, (16*60)//175 + 5)
        if hard == 3:
            delay = random.randint((32*60)//175 + 2, (32*60)//175 + 5)
        if hard == 5:
            delay = random.randint((48*60)//175 + 2, (48*60)//175 + 5)

        if lick == 1:
            songname = '"Mueva Los Huesos" by Gordon Goodwin'
        if lick == 2:
            songname = '"Machito" by Stan Kenton'
        if lick == 3:
            songname = '"Told You So" by Bill Holman'

        print('Playing: 🎶'+songname+'🎶')

        for x in range (1, delay):
            time.sleep(1)
            print('Counting Rests, please wait... (DO NOT PRESS ENTER)' )


        finished = newplay(hard, notes)
        #playnotes here

        print('Quickly copy the notes below! (There are '+str(finished.count(' '))+' Spaces!) (DO NOT PRESS ENTER)')
        print('COPY THIS:', finished, '|(ends here)|')
        _thread.start_new_thread(interrupt, (hard,))
        #Uses thread to run the interrupt function in the background so it can interrupt the input.
        play = input('TYPE HERE: ')
        while len(play) < 17:
            play += 'N'
        os.system('clear')
        #A feature that adds a character not in the original array "N."
        #This makes the finished user input just as long as the actual string,
        #so that the program doesn't try to access an index that doesn't exist because the string is too short.

        a = correct(finished, play, hard)
        b = rythym(finished, play, hard)
        # print(a)
        # print(b)
        percent = ((a/(len(finished)-finished.count(' '))*100))//1
        percent1 = ((b/finished.count(' '))*100)//1
        #Percent of notes and rhythm right.

        print('Notes: '+str(percent)+'% Correct')
        print('Rythym: '+str(percent1)+'% Correct')
        print('Listen to how you did! (DO NOT PRESS ENTER)')
        percenty = (percent+percent1)/2
    #Generates final average percent score.

        CopyOfPlayNotesWO = copy.deepcopy(playnoteswo)
        changed = changepitch(changerhythm(CopyOfPlayNotesWO, percent1/100), percent/100)
        pysynth.make_wav(changed, bpm = bpmval, pause = 0., boost = 1.1, fn = "song.wav", silent = True)
        s1.playFile("song.wav")


        if 90 <= percenty == 100:
            quote = 'This is very good, puppies! \n'
            for l in quote:
                sys.stdout.write(l)
                sys.stdout.flush()
                time.sleep(0.02)
            #Uses same code from game that prints more slowly. Used to emphasize moments of speech.

            self.nice = clamp(0, self.nice+5*hard, 10)
            self.humor = clamp(0, self.humor+2, 10)
            self.salty = clamp(0, self.salty-5, 10)
            self.mad = clamp(0, self.mad-7, 10)
        #Scoring above 90% improves your score.
        #Modulates based on difficuluty (Good score on hard level means greater mood improvement)
        elif 70 < percenty < 90:
            quote = 'Close, try again. You played this very well last rehersal! \n'
            for l in quote:
                sys.stdout.write(l)
                sys.stdout.flush()
                time.sleep(0.02)
            self.humor = clamp(0, self.humor+2, 10)
            self.salty = clamp(0, self.salty+2, 10)
        elif 50 < percenty <= 70:
            quote = 'There is a right way, and a wrong way. This is the wrong way. \n'
            for l in quote:
                sys.stdout.write(l)
                sys.stdout.flush()
                time.sleep(0.02)
            self.salty = clamp(0, self.salty+2, 10)
            self.mad = clamp(0, self.mad+2, 10)
            self.nice = clamp(0, self.nice-2, 10)
        elif 30 < percenty <= 50:
            quote = "Listen, there is no sight-reading in this orchestra.\n"
            quote1 = "You do this again, I wave goodbye to you.\n"
            for l in (quote, quote1):
                sys.stdout.write(l)
                sys.stdout.flush()
                time.sleep(0.02)
            self.salty = clamp(0, self.salty+3, 10)
            self.mad = clamp(0, self.mad+3, 10)
            self.humor = clamp(0, self.humor-3, 10)
            self.nice = clamp(0, self.nice-2, 10)
        elif percenty <= 30:
            quote = "Listen, this is completely wrong, your playing is all over the place!\n"
            quote1 = "What's wrong with this picture?\n"
            for l in (quote, quote1):
                sys.stdout.write(l)
                sys.stdout.flush()
                time.sleep(0.02)
            self.salty = clamp(0, self.salty+2, 10)
            self.mad = clamp(0, self.mad+5, 10)
            self.humor = clamp(0, self.humor-5, 10)
            self.nice = clamp(0, self.nice-5, 10)
        else:
            quote = "This is one of the greatest pieces in the history of jazz! You'll see! \n"
            for l in quote:
                sys.stdout.write(l)
                sys.stdout.flush()
                time.sleep(0.02)

        input('Press Enter to Continue: ')
        os.system('clear')


    def quote(self):
        if self.mad == 10:
            return ("LEAVE. The Choate Jazz Band has no place for unprepared musicians.")
        if self.nice >= 6 and self.mood >= 10:
            return ("Listen, this is the best you've played it! I'm serious!")
        if self.nice >= 6 and 5 <= self.mood < 10:
            return ("Bravissimo tutti! So very proud of you all.")
        if 5 < self.nice <= 6:
            return ("When you focus, you play this very well.")
        if self.salty >= 5 and self.mood >= 0:
            return ("You need to PRACTIIZE your music, ladies and gentlemen. You know who you are.")
        if self.salty >= 5 and 5 <= self.humor < 7:
            return ("Should I try it in Spanish? English is my best language...")
        if self.salty >= 5 and 8> self.humor >= 7 and self.mood >= -10:
            return ("Good morning!!! Are we having a Zen performance?")
        if self.salty >= 5 and 10 >= self.humor >= 8:
            return ("You need to practice with a little man on the Paris Subway...a METROGNOME!")
        if self.salty >= 5 and self.humor < 5:
            return ("Listen, I can't do your practicing for you. \nYou need to take this to your private teacher.")
        if self.salty >= 5 and 2 <= self.mad > 5:
            return ("There is no sight-reading in this orchestra. I can't help you if you do this.")
        if self.salty >= 5 and 2 <= self.mad < 5 and self.mood < -10:
            return ("This is NOT the football team. This is the Choate Jazz Band. FOCUS.")
        if 10 > self.salty >= 6 and 10 > self.mad >= 6:
            return ("You clearly haven't practiced this. You do this again, I wave goodbye to you.")
            #Varied "special quotes" that are triggered by certain mood states.

        else:
            return ('Please practice your music with sincerity of purpose and attention to detail.')
            #Default quote that applies across moods.



conductor = Ventre()
loop = True

while loop:
    os.system('clear')
    conductor.stats()
    action = input('What should Ventre do? (Type Quote, Conduct, or Nothing): ')
    #Continues to prompt user until "nothing" is inputted.

    if action.lower() == 'conduct':
        input('Press Enter To Play: ')
        print("Let's Play!")
        conductor.timing()
    #Initiates mood-game.
    elif action.lower() == 'quote':
        quote = '"'+str(conductor.quote())+'"\n'
        for l in quote:
            sys.stdout.write(l)
            sys.stdout.flush()
            time.sleep(0.02)
            #Uses same code from text game that prints more slowly. Used to emphasize moments of speech.

        time.sleep(0.5)
        input('Press Enter to Continue: ')
        os.system('clear')
        #Gives mood-determined quote.

    elif action.lower() == 'nothing':
        input('See you next rehearsal! (Press Enter)')
        os.system('clear')
        break
        #Leaves game by breaking loop.

    else:
        input("Ventre can't "+str(action)+"! (Press Enter)")
        os.system('clear')
        #Error-checking for input.

    loop = True
```

## Text-Based Game
> A Pokémon RPG with both choose-your-own adventure and battle aspects. The game includes several different features, including error checking loops, lowercase functions to make input easier, and room for choice. I wanted to make the game a really fun visual experience, so I incorporated scrolling text, clearing functions, and both adapted and original ASCII art. The art and the control flow of the battle sequence were definitely the hardest to debug and get right, but I'm really satisfied with how it came out.I learned a lot about using loops, incorporating functions, and programming logic/control flow during the project, and I worked really hard to really understand how each aspect of the program really works.

#### Text-Based Game Code:

```python
import time
import sys
import os
import random

#########################################FUNCTIONS#################################################

def health(x):
	block = ''
	for a in range(x):
		block += '█'
	return block
#Returns string based on variable x that holds number of health blocks

def bulbasaur(poke, hp, ehp, damage, edamage):
	intro = ('WEEDLE♂		    Lv4\n')
	intro0 = ('    hp: '+str(health(ehp))+'')
	intro1 = (' ('+str(ehp)+'/22)\n')
	intro2 = ('╚================================╝')
	intro3 = ('\n   _,           /|')
	intro4 = ("\n/     `.    /'_''`._	.__           ..:;:;::;:")
	intro5 = ("\n\_    ./   _\,   (__)	\ ''._    ..--''' '' ' ' '''--. ")
	intro6 = ("\n   `./-‘'/     `._./'   \.   '/' .   .'        '.   .`")
	intro7 = ("\n'_.-'   \_    ./'	\ | /    /            \   '.; '")
	intro8 = ("\n        `''-‘ 		,: /\ \.._\ __..===..__/_../ /`.")
	intro9 = ('\n			'+str(poke)+'♂		    Lv3')
	intro10 = ('\n			    hp: '+str(health(hp))+'')
	intro11 = (' ('+str(hp)+'/22)\n')
	intro12 = ('\n			╚================================╝')
	#I'm really proud of this ASCII art. I drew it myself and it took me a long time.
	for l in (intro, intro0, intro1, intro2, intro3, intro4, intro5, intro6, intro7, intro8, intro9, intro10, intro11, intro12):
		sys.stdout.write(l)
		sys.stdout.flush()
		time.sleep(0)
	#I left the sleep variable here (which controls the time between character printing) at zero 
	#so it looks like the screen updates almost instantly.

def charmander(poke, hp, ehp, damage, edamage):
	intro = ('WEEDLE♂		    Lv4\n')
	intro0 = ('    hp: '+str(health(ehp))+' ')
	intro1 = (' ('+str(ehp)+'/22)\n')
	intro2 = ('╚================================╝')
	intro3 = ('\n   _,           /|            _----_           `. (')
	intro4 = ("\n/     `.    /'_''`._	     .-__    '-.         ),'(,")
	intro5 = ("\n\_    ./   _\,   (__)	     |0.      '\.      (. )).)")
	intro6 = ("\n   `./-‘'/     `._./'        /_.      -  '\_.  /'`.(")
	intro7 = ("\n'_.-'   \_    ./'	     '-._    ___      /   ,|")
	intro8 = ("\n        `''-‘ 		         <\|'   --\  ..__ .'.")
	intro9 = ('\n			'+str(poke)+'♂		    Lv3')
	intro10 = ('\n			    hp: '+str(health(hp))+' ') 
	intro11 = (' ('+str(hp)+'/22)\n')
	intro12 = ('\n			╚================================╝')
	#I'm really proud of this ASCII art. I drew it myself and it took me a long time.
	for l in (intro, intro0, intro1, intro2, intro3, intro4, intro5, intro6, intro7, intro8, intro9, intro10, intro11, intro12):
		sys.stdout.write(l)
		sys.stdout.flush()
		time.sleep(0)

def fight():
	intro12 = ('\n╔================================╗')
	intro13 = ('\n	FIGHT		BAG')
	intro14 = ('\n')
	intro15 = ('\n	POKéMON		RUN')
	intro16 = ('\n╚================================╝')

	for l in (intro12, intro13, intro14, intro15, intro16):
		sys.stdout.write(l)
		sys.stdout.flush()
		time.sleep(0.03)

	choice_is_valid = False
	while not choice_is_valid:
		action = input('\nWhat will '+str(poke)+' do? ') 

		choice_is_valid = True
		if action.lower() == str('fight'):
			input("\nLet's go, "+str(poke)+"! (Press Enter)")
			os.system('clear')
		else:
			input('You must protect the professor! Fight! (Press Enter)')
			choice_is_valid = False
			os.system('clear')
	#Contains a loop for error checking.

def moveset1():
	intro12 = ('\n╔================================╗')
	intro13 = ('\n	TACKLE		GROWL')
	intro14 = ('\n')
	intro15 = ('\n	VINE WHIP	   ----')
	intro16 = ('\n╚================================╝')

	for l in (intro12, intro13, intro14, intro15, intro16):
		sys.stdout.write(l)
		sys.stdout.flush()
		time.sleep(0.03)

	choice_is_valid = False
	while not choice_is_valid:
		damage = 0
		action = input('\nWhat will '+str(poke)+' do? ') 

		choice_is_valid = True
		if action.lower() == str('tackle'):
			damage = random.randint(2, 6)
			if damage >= 4:
				print("\nCritical Hit!")
			input("\n"+str(poke)+" used Tackle! (Press Enter)")
			os.system('clear')
		elif action.lower() == str('growl'):
			damage = random.randint(4, 5)
			input("\n"+str(poke)+" used Growl! (Press Enter)")
			os.system('clear')
		elif action.lower() == str('vine whip'):
			damage = random.randint(6, 9)
			if damage >= 8:
				print("\nCritical Hit!")
			input("\n"+str(poke)+" used Vine Whip! (Press Enter)")
			os.system('clear')
		else:
			input('That\'s not a move! (Press Enter)')
			choice_is_valid = False
			os.system('clear')
	return damage
	#Contains a loop for error checking.
	#Many of these functions carry out both a side effect and return a variable, which is bad style.

def moveset2():
	damage = 0
	intro12 = ('\n╔================================╗')
	intro13 = ('\n	SCRATCH		GROWL')
	intro14 = ('\n')
	intro15 = ('\n	EMBER	    ----')
	intro16 = ('\n╚================================╝')

	for l in (intro12, intro13, intro14, intro15, intro16):
		sys.stdout.write(l)
		sys.stdout.flush()
		time.sleep(0.03)

	choice_is_valid = False
	while not choice_is_valid:
		action = input('\nWhat will '+str(poke)+' do? ') 

		choice_is_valid = True
		if action.lower() == str('scratch'):
			damage = random.randint(2, 5)
			if damage >= 4:
				print("\nCritical Hit!")
			input("\n"+str(poke)+" used Scratch! (Press Enter)")
			os.system('clear')
		elif action.lower() == str('growl'):
			damage = random.randint(4, 5)
			input("\n"+str(poke)+" used Growl! (Press Enter)")
			os.system('clear')
		elif action.lower() == str('ember'):
			damage = random.randint(6, 10)
			if damage >= 8:
				print("\nCritical Hit!")
			input("\n"+str(poke)+" used Ember! It's super effective! (Press Enter)")
			os.system('clear')
		else:
			input('That\'s not a move! (Press Enter)')
			choice_is_valid = False
			os.system('clear')
	return damage
	#Contains a loop for error checking.

def weedle():
	edamage = 0
	which = random.randint(1, 3)
	if which == 1:
		edamage = random.randint(2, 5)
		if edamage >= 4:
			print("\nCritical Hit!")
		input("\nWEEDLE used Poison Sting! (Press Enter)")
		os.system('clear')
	elif which == 2:
		edamage = random.randint(4, 5)
		input("\nWEEDLE used String Shot! (Press Enter)")
		os.system('clear')
	elif which == 3:
		edamage = random.randint(6, 9)
		if edamage >= 8:
			print("\nCritical Hit!")
		input("\nWEEDLE used Bug Bite! (Press Enter)")
		os.system('clear')
	return edamage
	#Contains a loop for error checking.

#########################################GAME START#################################################


intro1 = "                                  ,'\.\n"
intro2 = "    _.----.        ____         ,'  _\   ___    ___     ____\n"
intro3 = "_,-'       `.     |    |  /`.   \,-'    |   \  /   |   |    \  |`.\n"
intro4 = "\      __    \    '-.  | /   `.  ___    |    \/    |   '-.   \ |  |\n"
intro5 = " \.    \ \   |  __  |  |/    ,','_  `.  |          | __  |    \|  |\n"
intro6 = "   \    \/   /,' _`.|      ,' / / / /   |          ,' _`.|     |  |\n"
intro7 = "    \     ,-'/  /   \    ,'   | \/ / ,`.|         /  /   \  |     |\n"
intro8 = "     \    \ |   \_/  |   `-.  \    `'  /|  |    ||   \_/  | |\    |\n"
intro9 = "      \    \ \      /       `-.`.___,-' |  |\  /| \      /  | |   |\n"
intro10 = "       \    \ `.__,'|  |`-._    `|      |__| \/ |  `.__,'|  | |   |\n"
intro11 = "        \_.-'       |__|    `-._ |              '-.|     '-.| |   |\n"
intro12 = "                                `'  ーポケモンー              '-._|\n"

for l in (intro1, intro2, intro3, intro4, intro5, intro6, intro7, intro8, intro9, intro10, intro11, intro12):
	sys.stdout.write(l)
	sys.stdout.flush()
	time.sleep(0.05)
# I took inspiration for this "for" loop from a Stack Overflow article (in citations); it adds some delay between the text.

intro = "Welcome to Pokémon!\n"
for l in intro:
	sys.stdout.write(l)
	sys.stdout.flush()
	time.sleep(0.05)

input('(Press Enter to Continue)')

print('NOTE: While playing this game, please do not use the mouse!')
#Terminal keeps records of your running program, and only truly "clears" when you restart terminal. 
#The mouse would allow you to scroll back upwards to check previous parts of the game.
print('NOTE: Please click Enter only when asked!')
#I used an uncached input function as a way to stop the program until the player presses "Enter". 
#It can also proceed with a space, or any other input.
#The player repeatedly clearing the screen by skipping text is generally their fault, 
#and I could not find an effective solution to spamming.
name = input('Player Name: ')

input('(Press Enter to Continue)')
os.system('clear')
#Clearing the terminal!

print('Loading...', end='')
dots = "z Z z Z z Z\n"
for l in dots:
	sys.stdout.write(l)
	sys.stdout.flush()
	time.sleep(0.2)
#A programmed wait time for an authentic feel.

print('"Wake up, '+str(name)+'!"')
 
input('(Press Enter to Continue)')
os.system('clear')

intro1 = "You jolt awake from a terrible dream, grabbing wildly at the air.\n"
intro2 = "In it, you were being attacked by a giant monster!\n"
intro3 = "The alarm next to your bed lies silent.\n" 
intro4 = "Crap.\n" 
intro5 = "You forgot to set it.\n"
for l in (intro1, intro2, intro3, intro4, intro5):
	sys.stdout.write(l)
	sys.stdout.flush()
	time.sleep(0.08)

input('(Press Enter to Continue)')
os.system('clear')

intro1 = "You throw on clothes, grab your bag, and race out the door.\n"
intro2 = "Today is the day that you meet Professor Ki, a friend of your father’s.\n"
intro3 = "For some reason, he sent an important-looking letter asking to meet you.\n" 
for l in (intro1, intro2, intro3):
	sys.stdout.write(l)
	sys.stdout.flush()
	time.sleep(0.08)

choice = (input('Read the letter before you leave?(Y/N) '))

if choice.lower() == str('y') or choice.lower() == str('yes'):
#Removes case sensitivity.
	print('You read the letter.')
	print('    _________ 	Dear '+str(name)+',')
	intro1 = "   |\  slph /|		I was a good friend of your late father.\n"
	intro2 = "   | \     / |		Please meet me Friday at my Pokélab on Silph Road.\n"
	intro3 = "   |  `---'  | 	From,\n"
	intro4 = "   |__/___\__| 	Professor Ki (木教授)\n"

	for l in (intro1, intro2, intro3, intro4):
		sys.stdout.write(l)
		sys.stdout.flush()
		time.sleep(0.05)
#A good player will read the letter and find the professor's address! Or they can just guess.
input('(Press Enter to Continue)')
os.system('clear')

intro1 = "Panting, you watch the houses fly past you.\n"
intro2 = "As you stop to catch your breath, you realize something.\n"
intro3 = "Where exactly was Professor Ki’s office again?...\n" 
for l in (intro1, intro2, intro3):
	sys.stdout.write(l)
	sys.stdout.flush()
	time.sleep(1)
#A little more delay for suspenseful effect.

input('(Press Enter to Continue)')
os.system('clear')

intro1 = "As you scan the intersection, you see three road signs:\n" 
intro2 = "Nymph Road, Silph Road, and Victory Road.\n"
intro3 = "You rub your hair furiously, trying to remember which one led to the lab.\n"
for l in (intro1, intro2, intro3):
	sys.stdout.write(l)
	sys.stdout.flush()
	time.sleep(0.05)

choice_is_valid = False
while not choice_is_valid:
#Loop for error checking of correct variable. Worked with Ben Dreier to think up idea for this loop.
	intro1 = "	_________________________________________\n"
	intro2 = "	|	.-			    \.  |\n"
	intro3 = "	|	        VICTORY            -\   |\n"
	intro4 = "	|	           ^   		        |\n"
	intro5 = "	|			    	~._     |\n"
	intro6 = "	| NYMPH			          SILPH |\n"
	intro7 = "	|   <		                    >   |\n"
	intro8 = "	|  -_					|\n"
	intro9 = "	|	 \ (カゲドリ Intersection)--_,	|\n"
	intro10 = "	|____/__________________________________|\n"
	#Making ASCII art is fun.
	for l in (intro1, intro2, intro3, intro4, intro5, intro6, intro7, intro8, intro9, intro10):
		sys.stdout.write(l)
		sys.stdout.flush()
		time.sleep(0.05)

	choice = input('Choose a Road (Nymph, Silph, or Victory): ')
	os.system('clear')

	choice_is_valid = True
	if choice.lower() == str('silph') or choice == str('>'):
		print('You turn down Silph Road.')
		intro1 = "\nLuckily, your memory of the letter was right; you let out a sigh of relief.\n" 
		intro2 = "Tucked in a corner is a white, quaint-looking Pokélab.\n"
		for l in (intro1, intro2):
			sys.stdout.write(l)
			sys.stdout.flush()
			time.sleep(0.05)

		input("Enter Professor Ki's lab? (Press Enter)")
		os.system('clear')

		intro1 = "You enter quietly, not wanting to disturb.\n"
		intro2 = "However, as you walk hesitantly through the door, you see a thug with Pokémon\n" 
		intro3 = "threatening the professor! He is tied up with a sticky web of some kind.\n"
		intro4 = "You duck out of sight, luckily, only the professor sees you.\n"
		intro5 = "He gently kicks a metal case across the floor towards you.\n"
		for l in (intro1, intro2, intro3, intro4, intro5):
			sys.stdout.write(l)
			sys.stdout.flush()
			time.sleep(0.05)

		input("\nOpen case? (Press Enter)")
		os.system('clear')

		intro = "Inside the steel case are two Pokéballs! Which will you choose? \n"
		for l in intro:
			sys.stdout.write(l)
			sys.stdout.flush()
			time.sleep(0.02)

		choice_is_valid = False
		while not choice_is_valid:
		#Another use of the earlier style of the error checking loop.
			choice = input('\nChoose a Pokémon (Bulbasaur, Charmander): ')

			choice_is_valid = True
			if choice.lower() == str('charmander'):
				print('\nYou picked Charmander (ヒトカゲ) #004, a Fire Type!')
				poke = str('CHARMANDER')
			elif choice.lower() == str('bulbasaur'):
				print('\nYou picked Bulbasaur (フシギダネ) #001, a Grass Type!')
				poke = str('BULBASAUR')
			else:
				input("\nThat's not a Pokémon! Try again... (Press Enter)")
				choice_is_valid = False
				os.system('clear')

		intro1 = "\nThe thug hears the sliding of the case and turns towards you!\n" 
		intro2 = "Without thinking, you throw down the Pokéball, which releases a bright light.\n"
		for l in (intro1, intro2):
			sys.stdout.write(l)
			sys.stdout.flush()
			time.sleep(0.05)	

		input('(Press Enter to Continue)')
		os.system('clear')

		intro1 = "────────────────▄███████████▄────────────────\n"
		intro2 = "─────────────▄███▓▓▓▓▓▓▓▓▓▓▓███▄─────────────\n"
		intro3 = "────────────███▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓███────────────\n"
		intro4 = "───────────██▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓██───────────\n"
		intro5 = "──────────██▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓██──────────\n"
		intro6 = "─────────██▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓██─────────\n"
		intro7 = "────────██▓▓▓▓▓▓▓▓▓███████▓▓▓▓▓▓▓▓▓██────────\n"
		intro8 = "────────██▓▓▓▓▓▓▓▓██░░░░░██▓▓▓▓▓▓▓▓██────────\n"
		intro9 = "────────██▓▓▓▓▓▓▓██░░███░░██▓▓▓▓▓▓▓██────────\n"
		intro10 = "────────███████████░░███░░███████████────────\n"
		intro11 = "────────██░░░░░░░██░░███░░██░░░░░░░██────────\n"
		intro12 = "────────██░░░░░░░░██░░░░░██░░░░░░░░██────────\n"
		intro13 = "─────────██░░░░░░░░███████░░░░░░░░██─────────\n"
		intro14 = "──────────██░░░░░░░░░░░░░░░░░░░░░██──────────\n"
		intro15 = "───────────██░░░░░░░░░░░░░░░░░░░██───────────\n"
		intro16 = "────────────███░░░░░░░░░░░░░░░███────────────\n"
		intro17 = "─────────────▀███░░░░░░░░░░░███▀─────────────\n"
		intro18 = "────────────────▀███████████▀────────────────\n"

		for l in (intro1, intro2, intro3, intro4, intro5, intro6, intro7, intro8, intro9, intro10, intro11, intro12, intro13, intro14, intro15, intro16, intro17, intro18):
			sys.stdout.write(l)
			sys.stdout.flush()
			time.sleep(0.03)
		#Borrowed ASCII art cited at beginning.

		input("───Let's Battle! (Press Enter to Continue)───")
		os.system('clear')

		print('WEEDLE♂		    Lv4')
		print('    hp: ██████████████████████ ')
		print('╚================================╝')
		print('   _,           /|')
		print('/     `.    /"_""`._')
		print("\_    ./   _\,   (__)")
		print("   `./-‘'/     `._./'")
		print("'_.-'   \_    ./'")
		print("        `''-‘ ")
		print('			'+str(poke)+'♂		    Lv3')
		print('			    hp: ██████████████████████ ')
		print('			╚================================╝')
		print('╔================================╗')
		print('	FIGHT		BAG')
		print('')
		print('	POKéMON		RUN')
		print('╚================================╝')
		#ASCII art is fun.
		time.sleep(1)
		#Delay for effect.

		intro = '"Go, '+str(poke)+'!" \n'
		for l in intro:
			sys.stdout.write(l)
			sys.stdout.flush()
			time.sleep(0.02)

		time.sleep(2)
		os.system('clear')

		enemyhp = 22
		playerhp = 22
		dmg = 0
		edmg = 0
		winner = 1
		#Defining intial values to prevent errors. Health corresponds to number of blocks in health bar.
		#Damage corresponds to the amount dealt/subtracted from health each turn.

		if poke == str('BULBASAUR'):
			mon = 1
		else:
			mon = 2
		#Which Pokemon is being used?

		while enemyhp > 0 and playerhp > 0:
			if mon == 1:
				bulbasaur(poke, playerhp, enemyhp, dmg, edmg)
				fight()
				bulbasaur(poke, playerhp, enemyhp, dmg, edmg)
				dmg = moveset1()
				enemyhp -= dmg
				if enemyhp <= 0:
					winner = 1
					break
				bulbasaur(poke, playerhp, enemyhp, dmg, edmg)
				edmg = weedle()
				playerhp -= edmg
				if playerhp <= 0:
					winner = 0
				#Control Flow:
			#1) Prints Bulbasaur screen, including updating health variables.
			#2) Asks to fight (the only option), includes error checking loop. Clears.
			#3) Prints Bulbasaur screen again.
			#4) Asks for move, error checking loop, returns damage value "dmg".
			#5) Subtracts damage from enemy health.
			#6) Checks if enemy is dead, if so, exit loop and store winner as player.
			#7) Prints Bulbasaur screen, including updating health variables.
			#8) Random damage dealt by NPU.
			#9) Subtracts enemy damage from player health.
			#10) Checks if you are dead, if so, store winner as enemy.

			elif mon == 2:
				charmander(poke, playerhp, enemyhp, dmg, edmg)
				fight()
				charmander(poke, playerhp, enemyhp, dmg, edmg)
				dmg = moveset2()
				enemyhp -= dmg
				if enemyhp <= 0:
					winner = 1
					break
				charmander(poke, playerhp, enemyhp, dmg, edmg)
				edmg = weedle()
				playerhp -= edmg
				if playerhp <= 0:
					winner = 0
		
		if winner == 1:
			intro1 = "\nThe thug, defeated, angrily calls his Weedle back into its Pokéball.\n" 
			intro2 = 'He runs angrily towards you, screaming, "Darn you, kid!"\n'
			intro3 = '"BANG!"\n'
			intro4 = "To your surprise, he falls to the ground, unconcious.\n"
			intro5 = "A smiling Professor Ki stands behind him sheepishly, with a folding chair.\n"
			for l in (intro1, intro2, intro3, intro4, intro5):
				sys.stdout.write(l)
				sys.stdout.flush()
				time.sleep(0.05)	

			input('(Press Enter to Continue)')
			os.system('clear')

			intro1 = "\nLaughing, the professor thanks you.\n" 
			intro2 = '"Great work with '+poke+', '+name+'! Are you hurt?\n'
			intro3 = "You shake your head and begin to hand the Pokeball back, but he stops you.\n"
			intro4 = 'Keep it, '+name+'. You have the beginnings of a Pokemon Master, just like your father.\n'
			for l in (intro1, intro2, intro3, intro4):
				sys.stdout.write(l)
				sys.stdout.flush()
				time.sleep(0.05)	

			input('(Press Enter to Continue)')
			os.system('clear')

			intro1 = "                                  ,'\.\n"
			intro2 = "    _.----.        ____         ,'  _\   ___    ___     ____\n"
			intro3 = "_,-'       `.     |    |  /`.   \,-'    |   \  /   |   |    \  |`.\n"
			intro4 = "\      __    \    '-.  | /   `.  ___    |    \/    |   '-.   \ |  |\n"
			intro5 = " \.    \ \   |  __  |  |/    ,','_  `.  |          | __  |    \|  |\n"
			intro6 = "   \    \/   /,' _`.|      ,' / / / /   |          ,' _`.|     |  |\n"
			intro7 = "    \     ,-'/  /   \    ,'   | \/ / ,`.|         /  /   \  |     |\n"
			intro8 = "     \    \ |   \_/  |   `-.  \    `'  /|  |    ||   \_/  | |\    |\n"
			intro9 = "      \    \ \      /       `-.`.___,-' |  |\  /| \      /  | |   |\n"
			intro10 = "       \    \ `.__,'|  |`-._    `|      |__| \/ |  `.__,'|  | |   |\n"
			intro11 = "        \_.-'       |__|    `-._ |              '-.|     '-.| |   |\n"
			intro12 = "                                `'  ーポケモンー              '-._|\n"
			intro13 = "You win! Congratulations, "+name+"! Thanks for playing.\n"

			for l in (intro1, intro2, intro3, intro4, intro5, intro6, intro7, intro8, intro9, intro10, intro11, intro12, intro13):
				sys.stdout.write(l)
				sys.stdout.flush()
				time.sleep(0.05)

			input('(Press Enter to Continue)')
			os.system('clear')

		elif winner == 0:
			intro1 = "\nThe thug smiles, and grabs the briefcase of Pokemon.\n" 
			intro2 = '"Tough luck, kid," he smirks.\n'
			intro3 = "His Pokemon traps you in a web of sticky material, before he runs out the door.\n"
			intro4 = "Game Over.\n"
			for l in (intro1, intro2, intro3, intro4):
				sys.stdout.write(l)
				sys.stdout.flush()
				time.sleep(0.05)

			input("\nTry again next time... (Press Enter)")
			os.system('clear')

	elif choice.lower() == str('nymph') or choice == str('<'):
		print('You turn down Nymph Road.')

		intro1 = "After two hours of walking, you are extremely tired. Why didn't you check the letter?\n" 
		intro2 = "There's no way you'll ever make it to the professor's office.\n"
		intro3 = "Game Over.\n"
		for l in (intro1, intro2, intro3):
			sys.stdout.write(l)
			sys.stdout.flush()
			time.sleep(0.05)
		input("\nYou chose wrong...play again...(Press Enter)")
		os.system('clear')

	elif choice.lower() == str('victory') or choice == str('^'):
		print('You turn down Victory Road.')
		intro1 = "After two hours of walking, you are extremely tired. Why didn't you check the letter?\n" 
		intro2 = "There's no way you'll ever make it to the professor's office.\n"
		intro3 = "Game Over.\n"
		for l in (intro1, intro2, intro3):
			sys.stdout.write(l)
			sys.stdout.flush()
			time.sleep(0.05)
		input("\nYou chose wrong...play again...(Press Enter)")
		os.system('clear')

	else:
		input("\nThat's not a road! Try again... (Press Enter)")
		choice_is_valid = False
		os.system('clear')
```
