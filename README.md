Demo
====
# Mini-project #6 - Blackjack

import simplegui
import random

# load card sprite - 949x392 - source: jfitz.com
CARD_SIZE = (73, 98)
CARD_CENTER = (36.5, 49)
card_images = simplegui.load_image("http://commondatastorage.googleapis.com/codeskulptor-assets/cards.jfitz.png")

CARD_BACK_SIZE = (71, 96)
CARD_BACK_CENTER = (35.5, 48)
card_back = simplegui.load_image("http://commondatastorage.googleapis.com/codeskulptor-assets/card_back.png")    

# initialize some useful global variables
in_play = False
outcome = ""
score = 0
points = ""

# define globals for cards
SUITS = ('C', 'S', 'H', 'D')
RANKS = ('A', '2', '3', '4', '5', '6', '7', '8', '9', 'T', 'J', 'Q', 'K')
VALUES = {'A':1, '2':2, '3':3, '4':4, '5':5, '6':6, '7':7, '8':8, '9':9, 'T':10, 'J':10, 'Q':10, 'K':10}


class Card:
    def __init__(self, suit, rank):
        if (suit in SUITS) and (rank in RANKS):
            self.suit = suit
            self.rank = rank
        else:
            self.suit = None
            self.rank = None
            print "Invalid card: ", suit, rank

    def __str__(self):
        return self.suit + self.rank

    def get_suit(self):
        return self.suit

    def get_rank(self):
        return self.rank

    def draw(self, canvas, pos):
        card_loc = (CARD_CENTER[0] + CARD_SIZE[0] * RANKS.index(self.rank), 
                    CARD_CENTER[1] + CARD_SIZE[1] * SUITS.index(self.suit))
        canvas.draw_image(card_images, card_loc, CARD_SIZE, [pos[0] + CARD_CENTER[0], pos[1] + CARD_CENTER[1]], CARD_SIZE)
        
class Hand:
    def __init__(self):
        self.hand = []

    def __str__(self):
        ans = " "
        for i in range(len(self.hand)):
            ans += str(self.hand[i]) + " "
        return ans

    def add_card(self, card):
        self.hand.append(card)
        return self.hand

    def get_value(self):
        self.hand_value = 0
        for i in self.hand:
            self.hand_value += VALUES[i.get_rank()]
        for card in self.hand:
            
            # decides whether "A" should be counted as 1 or 11 
            if card.get_rank() == "A":
                self.hand_value += 10
                if self.hand_value > 21:
                   self.hand_value -= 10
                else:
                   self.hand_value = self.hand_value
        return self.hand_value
   
    def draw(self, canvas, pos):
        for card in self.hand:
            pos = (pos[0] + 80, pos[1])
            card.draw(canvas,pos)

class Deck:
    def __init__(self):
        self.deck = []
        for i in SUITS:
            for j in RANKS:
                self.card = Card (i, j)
                self.deck.append(self.card)

    def shuffle(self):
        self.deck.append(self.card)
        random.shuffle(self.deck)

    def deal_card(self):
        return self.deck.pop()
    
    def __str__(self):
        ans = " "
        for i in range(len(self.deck)):
           ans +=" " + str(self.deck[i])
        return ans

deck = Deck ()
player_hand = Hand ()
dealer_hand = Hand ()

def deal():
    global outcome, in_play, deck, player_hand, dealer_hand, points, score
    deck = Deck()
    deck.shuffle()
    player_hand = Hand()
    dealer_hand = Hand()
    
    # deals two cards to the dealer and the player at the beginning of a round
    player_hand.add_card(deck.deal_card())
    player_hand.add_card(deck.deal_card())
    dealer_hand.add_card(deck.deal_card())
    dealer_hand.add_card(deck.deal_card())
    outcome = "You have " + str(player_hand.get_value()) + ". Hit or stand?"
    points = "score: " + str(score)

    # deducts score if Deal button was pressed in the middle of the round
    if in_play:
        outcome = "'Deal' button was pressed in the middle of a round!"
        score -= 1
        in_play = False
        points = "score: " + str(score)
    in_play = True

def hit():
    global outcome, in_play, deck, player_hand, dealer_hand, score, points
    # based on players hand value, suggest to hit or to stand
    if player_hand.get_value() <= 21 and in_play:   # if player hand less than 21, deals a card
        player_hand.add_card(deck.deal_card())
        if player_hand.get_value()<= 21:
            outcome = "You have " + str(player_hand.get_value()) + ". Hit or stand?"
        elif player_hand.get_value() > 21:          # notifies the player that he went busted
            outcome = "You have busted! Dealer has won! New deal?"
            score -= 1
            in_play = False
            points = "score: " + str(score)
       
def stand():
    global outcome, in_play, deck, player_hand, dealer_hand, score, points

    # checks if player had busted
    if player_hand.get_value() > 21:
        canvas.draw_text (outcome, [250, 50], 40, "Black")
        print outcome
        in_play = False

    # deals cards to the dealer and determines the winner    
    elif player_hand.get_value() <= 21:
        while dealer_hand.get_value() <= 17:
            dealer_hand.add_card(deck.deal_card())
        if dealer_hand.get_value() > 21:       # checks if dealer has busted
            outcome = "Dealer has busted! You have won! New deal?"
            in_play = False
            score += 1
            points = "score: " + str(score)
        elif dealer_hand.get_value() <= 21:     # determines the winner
            if dealer_hand.get_value() >= player_hand.get_value():
                outcome = "Dealer has won! New deal?"
                in_play = False  
                score -= 1
                points = "score: " + str(score)
            else:
                outcome = "You have won! New deal?"
                in_play = False
                score += 1
                points = "score: " + str(score)


# draw handler    
def draw(canvas):
    global outcome, in_play, deck, player_hand, dealer_hand, score

    player_hand.draw(canvas, [60, 100])
    dealer_hand.draw(canvas, [60, 400])
    canvas.draw_text("Blackjack", [100, 50], 40, "Blue")
    canvas.draw_text(outcome, [30, 340], 30, "Black")
    canvas.draw_text(points, [500, 50], 40, "White")
    canvas.draw_text("Player", [150, 250], 35, "Olive")
    canvas.draw_text("Dealer", [150, 550], 35, "Olive")
    if in_play:
       canvas.draw_image(card_back, CARD_BACK_CENTER, CARD_BACK_SIZE, [140 + CARD_BACK_CENTER[0], 400 + CARD_BACK_CENTER[1]], CARD_BACK_SIZE)


# initialization frame
frame = simplegui.create_frame("Blackjack", 700, 600)
frame.set_canvas_background("Green")

#create buttons and canvas callback
frame.add_button("Deal", deal, 200)
frame.add_button("Hit",  hit, 200)
frame.add_button("Stand", stand, 200)
frame.set_draw_handler(draw)


# get things rolling
frame.start()


