# Garblanga

We can model a hand of cards in the well-known card game, Garblanga, as a graph, where each card is a node, and there is an undirected edge between two cards if they have the same suit or number. Then, the problem of determining whether a given hand of cards constitutes a valid sequence is really just checking if all the nodes in the graph are reachable from each other. Running either breadth-first or depth-first search from any one node and keeping track of which nodes have been visited will let us know whether all the nodes are reachable, and thus whether a hand is a valid sequence.

# Solution

```python3
powers = {
    "2": 2,
    "3": 3,
    "4": 4,
    "5": 5,
    "6": 6,
    "7": 7,
    "8": 8,
    "9": 9,
    "10": 10,
    "A": 11,
    "J": 12,
    "Q": 13,
    "K": 14,
}

def can_connect(hand_card, card):
    return hand_card[0] == card[0] or hand_card[1] == card[1]

def can_win(hand, cards_left):
    if len(cards_left) == 0:
        return True
    
    win = False
    for card in cards_left:
        if hand == [] or any([can_connect(hand_card, card) for hand_card in hand]):
            win = win or can_win(hand + [card], [card_left for card_left in cards_left if card_left != card])
    return win

cases = int(input())
for _ in range(cases):
    hand = [(powers[x[:-1]], x[-1:]) for x in input().split(" ")]
    if can_win([], hand):
        print("win")
    else:
        print("draw")
    
```