import nltk
from nltk.tokenize import PunktSentenceTokenizer
from tabulate import tabulate
import matplotlib
from nltk import Tree

appendix_text = """The Natural Programming Project will study ways to make learning how to program significantly easier. More people will be able to create useful, interesting, and sophisticated programs. The goal is to study how nonprogrammers reason about programming concepts, then to create one or more new programming languages and environments that apply these findings.\n
For example, the new Java and JavaScript languages use the same mechanisms for looping, conditionals, and assignments that have been shown to cause many errors for both beginning and expert programmers in the C language.\n
A thorough investigation of the ESP and HCI literature revealed many results that can be used to guide the design of a new programming system. However, there are many significant "holes" in the knowledge about how people reason about programs and programming. We need to study about fundamental paradigms of computing are the most natural. Many new systems are object oriented (C++, Java), but systems aimed at novice programmers tend to be event based instead (Visual Basic, HyperTalk), and the language research community is studying functional styles. The old imperative style is easiest for beginning programmers. We will perform user studies to investigate this question."""

arr = []#array containing information from part one

def prompt_main():
    option = -1
    while not 0 <= option <= 2:
      print('Main Menu')
      print('0 Exit Program.')
      print('1 Adjust or view the text.')
      print('2 Run Part I')
      try:
        option = int(input('What would you like to do? '))
      except:
        continue
      if option == 0:
        quit()
      else:
        return option

def prompt_count():
    count = -1
    while not 0 <= count:
      try:
        count = int(input('Please enter a non-negative integer. '))
      except:
        continue
    return count
  
def prompt_text():
    option = 7
    while 1 <= option <= 7:
      print('Text Menu')
      print('0 Return to Main Menu')
      print('1 Replace a sentence')
      print('2 Repalce a paragraph')
      print('3 Remove a sentence')
      print('4 Remove a paragraph')
      print('5 Insert a sentence')
      print('6 Insert a paragraph')
      print('7 View the text')
      try:
        option = int(input('What would you like to do? '))
      except:
        continue
      if option == 0:
        prompt_main()
      elif option == 1:
        sent_num = int(input('Which sentence would you like to replace? '))
        sent = input('What would you like the new sentence to be? ')
        print(adjust_sentence(appendix_text,sent_num,sent,option))
      elif option == 2:
        para_num = int(input('Which paragraph would you like to replace? '))
        para = input('What would you like the new paragraph to be?')
        print(adjust_paragraph(appendix_text,para_num,para,option))
      elif option == 3:
        sent_num = int(input('Which sentence would you like to remove? '))
        print(adjust_sentence(appendix_text,sent_num,None,option))
      elif option == 4:
        para_num = int(input('Which paragraph would you like to remove? '))
        print(adjust_paragraph(appendix_text,para_num,None,option))
      elif option == 5:
        sent_num = int(input('After which sentence would you like to insert your new sentence? '))
        sent = input('What would you like the new sentence to be? ')
        print(adjust_sentence(appendix_text,sent_num,sent,option))
      elif option == 6:
        sent_num = int(input('After which paragraph would you like to insert your new paragraph? '))
        sent = input('What would you like the new paragraph to be? ')
        print(adjust_paragraph(appendix_text,para_num,para,option))
      elif option == 7:
        print(appendix_text)
        
def obj_table(text):
  tokens = nltk.word_tokenize(text)
  tag = nltk.pos_tag(tokens)
  paragraphs = text.split("\n\n")
  sentence = 1
  for par in range(len(paragraphs)): #count paragraphs
    verb_object = find_verb_object(paragraphs[par])
    tdarr = [None] * 4
    for i in range(len(verb_object)):
      ind = 0
      if verb_object[i] == '.':
        sentence += 1
      elif verb_object[i] == ',' or verb_object[i][0] == '(' or verb_object[i][0] == ')':
        pass
      else:
        word = verb_object[i].split()
        if len(word) == 1: 
          tdarr[0] = par + 1
          tdarr[1] = sentence
          tdarr[2] = verb_object[i]
          tdarr[3] = verb_object[i+1]
        else:
          tdarr[0] = par + 1
          tdarr[1] = sentence
          tdarr[2] = word[0]
          tdarr[3] = str(word[1:len(word)]).replace("['", "").replace("']", "").replace ("'","").replace(",","")
        arr.append(tdarr)
        tdarr = [None] * 4
  return tabulate(arr, headers = ["Para #", "Sent #", "Verb", "Object"])

def find_verb_object(text):
  custom_sent_tokenizer = PunktSentenceTokenizer(text)
  tokenize = custom_sent_tokenizer.tokenize(text)
  for i in tokenize:
    tokens = nltk.word_tokenize(text)
    tag = nltk.pos_tag(tokens)
    grammar = r"""
    INF:{<TO><NN>}
    AP:{<JJ><.><JJ><.><CC><JJ>|<JJ>*|<RB><JJ.>|<JJ.><JJ>|<JJ>}
    NP:{<DT><AP><NN>*|<DT><RBS><AP>|<DT><AP><NN>|<VBG><AP><NNS>|<VBG>|<NN.>|<DT><NN>|<DT><NN.>*|<NP>*}
    PP:{<IN><NN.>|<IN><NN>|<IN><VBG>|<IN><NP>}
    VP:{<VB><CD><CC><JJR><AP><NN><NP><CC><NP>|<VBD><AP><NP>|<VBP><AP><VBN>|<VBZ><JJS><PP><NP>|<VBN><PP><NP>|<VB><AP><NP>|<NN><PP><NP>|<NN><PP><CC><NP>|<VB><IN><AP><NN>|<VB><WRB><NP>|<VB><NP>|<VB.><NP>|<INF><NP>}
    MoreVP:{<VBN><TO><VP>|<VBN><TO><VP><NN><PP><NP>|<VBP><TO><VP>|<VB><AP><TO><VP>|<VBZ><TO><VP>}
    PER:{<.>}"""
    chunkParser=nltk.RegexpParser(grammar)
    chunked = chunkParser.parse(tag)
  objs=[]
  for subtree3 in chunked.subtrees():
    if subtree3.label() == 'MoreVP' or subtree3.label() == 'VP' or subtree3.label() == 'PER':
      objs.append(" ".join([a for (a,b) in subtree3.leaves()]))
  return objs 
  
def part_two(arr):
  verbs =[None]*(len(arr)+1)
  chart =[None]*(len(arr)+1)
  for i in range(len(arr)):
    verbs[i+1]=arr[i][3]
  verb = set(verbs)
  verbs = list(verb)
  chart[0] = verbs
  verbs =[None]*(len(arr)+1)
  for i in range(len(arr)):
    verbs[0] = arr[i][2]
    chart[i+1] = verbs
    verbs =[None]*len(arr)
  ind = 1
  return tabulate(chart)
    
def adjust_sentence(text, num, sentence, option):
  to_adjust = nltk.sent_tokenize(text)
  num -= 1
  if option == 1:
    to_adjust[num] = sentence
  elif option == 3:
    new_array = []
    for i in range(len(to_adjust)):
      if i == num:
        pass
      else:
        new_array.append(to_adjust[i])
    to_adjust = new_array
  elif option == 5:
    new_array = []
    for i in range(len(to_adjust)):
      if i == num:
        new_array.append(sentence)
      else:
        new_array.append(to_adjust[i])
  appendix_text = str(to_adjust).replace("[","").replace("]","").replace("'","").replace(",","")
  return appendix_text

def adjust_paragraph(text, num, para, option):
  to_adjust = text.split("\n\n")
  num -= 1
  if option == 2:
    to_adjust[num] = para
  elif option == 4:
    new_array = []
    for i in range(len(to_adjust)):
      if i == num:
        pass
      else:
        new_array.append(to_adjust[i])
    to_adjust = new_array
  elif option == 6:
    new_array = []
    for i in range(len(to_adjust)):
      if i == num:
        new_array.append(para)
      else:
        new_array.append(to_adjust[i])
  appendix_text = str(to_adjust)
  str(appendix_text).replace("[ ","").replace(" ]","").replace("'","").replace(",","")
  return appendix_text

choice = 1
while choice != 0:
  choice = prompt_main()
  if choice == 1:
    prompt_text()
  elif choice == 2:
    print(obj_table(appendix_text))
