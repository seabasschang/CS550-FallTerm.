# CS550 Fall Term Work

## Ventre Program
> Using core concepts from the term to create a class-based representation of the loveable Choate Jazz Band director Phil Ventre, who gives iconic quotes based on his mood. Allows interaction with "Ventre" through a typing game of variable difficulty that can change his mood and then show the user how they did by playing a song (Mueva Los Huesos, Machito, or Told You So) adjusted to reflect how well they did based on a tuning and rythym factor. 

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



