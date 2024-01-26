#!/bin/bash

# Function to create a grid with random values and set one index to 0
create_grid() {
  # Define the grid size
  rows=5
  columns=5

  # Define the index to set to 0 (e.g., the first index, 0-based)
  index_to_set_to_zero=0

  # Loop through each row
  for ((i=0; i<rows; i++)); do
    # Loop through each column
    for ((j=0; j<columns; j++)); do
      # Generate a random value between 1 and 25
      value=$(( (RANDOM % 25) + 1 ))

      # If this is the index to set to 0, make it 0
      if ((i * columns + j == index_to_set_to_zero)); then
        value=0
      fi

      # Store the value in the grid array
      grid[i * columns + j]=$value
      printf "%2d " $value
    done
    echo
  done
}

# Function to create a grid with a specific goal state
goal() {
  # Define the desired goal state (you can customize this)
  local rows=5
  local columns=5
  local goal_state=(
    1  2  3  4  5
    6  7  8  9 10
   11 12 13 14 15
   16 17 18 19 20
   21 22 23 24  0
  )

  # Loop through each row
  for ((i=0; i<rows; i++)); do
    # Loop through each column
    for ((j=0; j<columns; j++)); do
      # Get the value from the goal state
      local value=${goal_state[$i * columns + $j]}

      # Store the value in the goal array
      goal_grid[i * columns + j]=$value
    done
  done
}

# Function to count the number of inversions in a given state
count_inversions() {
  local -n state=$1
  local n=${#state[@]}
  local inversions=0

  for ((i = 0; i < n - 1; i++)); do
    for ((j = i + 1; j < n; j++)); do
      if ((state[i] != 0 && state[j] != 0 && state[i] > state[j])); then
        ((inversions++))
      fi
    done
  done

  echo $inversions
}

# Function to check if a puzzle with the given initial and goal states is solvable
is_solvable() {
  local -n initial_state=$1
  local -n goal_state=$2
  local n=${#initial_state[@]}

  local initial_inversions=$(count_inversions initial_state)
  local goal_inversions=$(count_inversions goal_state)

  if ((n % 2 == 1)); then
    # Odd-sized puzzle
    if ((initial_inversions % 2 == goal_inversions % 2)); then
      echo "Solvable"
    else
      echo "Not solvable"
    fi
 fi
}

# Function to find legal moves for a given state
legal_moves() {
  local -n state=$1
  local rows=5
  local columns=5

  for ((i = 0; i < rows; i++)); do
    for ((j = 0; j < columns; j++)); do
      local index=$((i * columns + j))
      if ((state[index] == 0)); then
        # This is the blank (0) tile, find adjacent legal moves
        local moves=()
        if ((i > 0)); then
          moves+=("U")  # Up
        fi
        if ((i < rows - 1)); then
          moves+=("D")  # Down
        fi
        if ((j > 0)); then
          moves+=("L")  # Left
        fi
        if ((j < columns - 1)); then
          moves+=("R")  # Right
        fi
        echo "${moves[@]}"
        return
      fi
    done
  done
}

# Function to make a move in the state
make_move() {
  local -n state=$1
  local move=$2
  local rows=5
  local columns=5

  local blank_i=-1
  local blank_j=-1

  for ((i = 0; i < rows; i++)); do
    for ((j = 0; j < columns; j++)); do
      local index=$((i * columns + j))
      if ((state[index] == 0)); then
        blank_i=$i
        blank_j=$j
        break
      fi
    done
    if ((blank_i != -1)); then
      break
    fi
  done

  if [[ "$move" == "U" && $blank_i -gt 0 ]]; then
    local source_index=$((blank_i * columns + blank_j))
    local target_index=$((source_index - columns))
    local tmp=${state[source_index]}
    state[source_index]=${state[target_index]}
    state[target_index]=$tmp
  elif [[ "$move" == "D" && $blank_i -lt $((rows - 1)) ]]; then
    local source_index=$((blank_i * columns + blank_j))
    local target_index=$((source_index + columns))
    local tmp=${state[source_index]}
    state[source_index]=${state[target_index]}
    state[target_index]=$tmp
  elif [[ "$move" == "L" && $blank_j -gt 0 ]]; then
    local source_index=$((blank_i * columns + blank_j))
    local target_index=$((source_index - 1))
    local tmp=${state[source_index]}
    state[source_index]=${state[target_index]}
    state[target_index]=$tmp
  elif [[ "$move" == "R" && $blank_j -lt $((columns - 1)) ]]; then
    local source_index=$((blank_i * columns + blank_j))
    local target_index=$((source_index + 1))
    local tmp=${state[source_index]}
    state[source_index]=${state[target_index]}
    state[target_index]=$tmp
  fi
  
  # Save the move to the moves array
  moves+=("$move")
}

print_grid() {
  local -n state=$1
  local rows=5
  local columns=5

  for ((i = 0; i < rows; i++)); do
    for ((j = 0; j < columns; j++)); do
      local index=$((i * columns + j))
      printf "%2d " "${state[index]}"
    done
    echo
  done
}

# Function to print the sequence of moves
print_moves() {
  for move in "${moves[@]}"; do
    if [[ "$move" == "U" ]]; then
      echo "Move Up"
    elif [[ "$move" == "D" ]]; then
      echo "Move Down"
    elif [[ "$move" == "L" ]]; then
      echo "Move Left"
    elif [[ "$move" == "R" ]]; then
      echo "Move Right"
    fi
  done
  echo "Reached the goal state."
}

# Function to check if the given state matches the goal state
is_goal() {
  local -a state=("${!1}")  # Convert the variable name to an array
  local -a goal_state=("${!2}")  # Convert the variable name to an array
  local n=${#state[@]}

   for ((i = 0; i < n; i++)); do
    if ((state[i] != goal_state[i])); then
      echo "false"
      return
    fi
  done

  echo "true"
}

# Create the initial and goal states
create_grid
goal

# Check if the puzzle with the given states is solvable
is_solvable grid goal_grid

if [[ "$(is_solvable grid goal_grid)" == "Not solvable" ]]; then
  echo "This puzzle is not solvable. Quitting."
  exit 1
fi

moves=()  # Initialize moves array to store the sequence of moves

while true; do
  echo "Enter a move (U/D/L/R to move the blank tile, 'Q' to quit):"
  read move

  if [[ "$move" == "Q" ]]; then
    echo "Quitting."
    break
  fi

  if [[ "$move" =~ ^[UDLR]$ ]]; then
    legal_moves=($(legal_moves grid))
    if [[ " ${legal_moves[@]} " =~ " $move " ]]; then
      make_move grid "$move"
      echo "Updated Grid:"
      print_grid grid

      # Check if the goal state has been reached
      if [[ "$(is_goal grid goal_grid)" == "true" ]]; then
        echo "Congratulations! You've reached the goal state."
        print_moves  # Print the sequence of moves
        break
      fi
    else
      echo "Invalid move. Please enter a valid move (U/D/L/R) or 'Q' to quit."
    fi
  else
    echo "Invalid input. Please enter a valid move (U/D/L/R) or 'Q' to quit."
  fi
done
