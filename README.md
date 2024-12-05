# Wordle-Scoreboard
# 3-Player Wordle Scoreboard Tracker
# Writen in Python for PyCharm

import json
import os

# Player names
players = ["Drew", "Katie", "Chris"]

# Initialize game data
game_data = {
    "players": {player: {"wins": 0, "losses": 0, "ties": 0, "total_guesses": 0} for player in players},
    "head_to_head": {tuple(sorted([p1, p2])): {"wins": {p1: 0, p2: 0}, "ties": 0} for p1 in players for p2 in players if
                     p1 != p2},
    "total_games": 0,
    "round_history": []  # Store scores for each round
}

# File location for saving and loading scores
score_file = os.path.join(os.getcwd(), "wordle_scores.json")

# Load existing scores from file if available
def load_scores():
    if os.path.exists(score_file):
        try:
            with open(score_file, "r") as file:
                data = json.load(file)
                game_data.update(data)  # Update game data with loaded data
                print("Scores loaded successfully.")
        except json.JSONDecodeError:
            print("Error: JSON file is corrupted. Starting fresh.")
        except Exception as e:
            print(f"Error loading scores: {e}")
    else:
        print("No existing score file found. Starting fresh.")

# Save scores to file
def save_scores():
    try:
        with open(score_file, "w") as file:
            json.dump(game_data, file, indent=4)
        print("Scores saved successfully.")
    except Exception as e:
        print(f"Error saving scores to the file: {e}")

# Update head-to-head record between two players
def update_head_to_head(player1, player2, winner):
    pair = tuple(sorted([player1, player2]))
    if winner == "tie":
        game_data["head_to_head"][pair]["ties"] += 1
    else:
        game_data["head_to_head"][pair]["wins"][winner] += 1

# Update main player scores
def update_scores(guesses_dict):
    game_data["total_games"] += 1

    # Add current round's guesses to the round history
    game_data["round_history"].append(guesses_dict)

    # Determine round outcomes and update player stats
    for player1 in players:
        for player2 in players:
            if player1 != player2:
                if guesses_dict[player1] < guesses_dict[player2]:
                    update_head_to_head(player1, player2, player1)
                    game_data["players"][player1]["wins"] += 1
                    game_data["players"][player2]["losses"] += 1
                elif guesses_dict[player1] > guesses_dict[player2]:
                    update_head_to_head(player1, player2, player2)
                    game_data["players"][player1]["losses"] += 1
                    game_data["players"][player2]["wins"] += 1
                else:
                    update_head_to_head(player1, player2, "tie")
                    game_data["players"][player1]["ties"] += 1
                    game_data["players"][player2]["ties"] += 1

    # Update total guesses for each player
    for player, guesses in guesses_dict.items():
        game_data["players"][player]["total_guesses"] += guesses

# Input validation for guesses
def get_int_input(prompt):
    while True:
        try:
            return int(input(prompt))
        except ValueError:
            print("Please enter a valid number.")

# Print current scores
def print_scores():
    print("\nSpelling B's Scoreboard:\n")
    print("Current Scores:")
    for player in players:
        stats = game_data["players"][player]
        print(
            f"{player:10} | Wins: {stats['wins']} | Losses: {stats['losses']} | Ties: {stats['ties']} | Total Guesses: {stats['total_guesses']}"
        )

    print("\nHead-to-Head Records:")
    for pair, stats in game_data["head_to_head"].items():
        print(
            f"{pair[0]:10} vs {pair[1]:10} | {pair[0]} Wins: {stats['wins'][pair[0]]} | {pair[1]} Wins: {stats['wins'][pair[1]]} | Ties: {stats['ties']}"
        )

    print(f"\nTotal Games Played: {game_data['total_games']}")

    print("\nRound History:")
    for i, round_data in enumerate(game_data["round_history"], 1):
        print(f"Round {i}: {round_data}")

# Main game loop
def main():
    load_scores()

    while True:
        print("\n--- Spelling B's Scoreboard ---")
        print("1. Enter guesses for this round")
        print("2. Reset scores to default")
        print("3. Exit")
        choice = input("Enter your choice: ")

        if choice == "1":
            guesses_dict = {}
            for player in players:
                guesses = get_int_input(f"Enter the number of guesses for {player}: ")
                guesses_dict[player] = guesses
            update_scores(guesses_dict)
            print_scores()
            save_scores()

        elif choice == "2":
            game_data["players"] = {player: {"wins": 0, "losses": 0, "ties": 0, "total_guesses": 0} for player in players}
            game_data["head_to_head"] = {tuple(sorted([p1, p2])): {"wins": {p1: 0, p2: 0}, "ties": 0} for p1 in players
                                         for p2 in players if p1 != p2}
            game_data["total_games"] = 0
            game_data["round_history"] = []
            print("Scores have been reset.")
            save_scores()

        elif choice == "3":
            print("Exiting the game.")
            save_scores()
            break

        else:
            print("Invalid choice. Please try again.")

if __name__ == "__main__":
    main()
