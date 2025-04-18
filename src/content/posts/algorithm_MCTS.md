---
title: algorithm_MCTS
published: 2025-04-18
description: ''
image: ''
tags: []
category: ''
draft: false 
lang: ''
---


<iframe width="100%" height="500px" src="https://godbolt.org/e#g:!((g:!((g:!((h:codeEditor,i:(filename:'1',fontScale:14,fontUsePx:'0',j:1,lang:c,selection:(endColumn:1,endLineNumber:1,positionColumn:1,positionLineNumber:1,selectionStartColumn:1,selectionStartLineNumber:1,startColumn:1,startLineNumber:1),source:''),l:'5',n:'0',o:'C+source+%231',t:'0')),k:50,l:'4',n:'0',o:'',s:0,t:'0'),(g:!((h:executor,i:(argsPanelShown:'1',compilationPanelShown:'0',compiler:cg132,compilerName:'',compilerOutShown:'0',execArgs:'',execStdin:'',fontScale:14,fontUsePx:'0',j:1,lang:c,libs:!(),options:'',source:1,stdinPanelShown:'1',tree:'1',wrap:'1'),l:'5',n:'0',o:'Executor+x86-64+gcc+13.2+(C,+Editor+%231)',t:'0')),k:50,l:'4',n:'0',o:'',s:0,t:'0')),l:'2',n:'0',o:'',t:'0')),version:4"></iframe>

Today is about MCTS. Monte Carlo Tree Search. 

First, let's recover the memory of the MCTS. Its algorithm is as follows:

$$
V(s) = \frac{\sum_{i=1}^{N} R_i}{N}
$$

where:
- $$V(s) $$is the value of state $$s$$,
- $$R_i$$ is the reward received from the $$i$$-th simulation,
- $$N$$ is the number of simulations from state $$s$$.

so, the Monte Carlo Tree Search is an algorithm with the idea of statistics. 

```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
int main() {
    long point_inside_circle = 0;
    long i;
    double x, y;
    double distance_squared;
    double radius_squared = 1.0;
    int num_points = 10000000;
    double pi_estimate;
    srand(time(NULL));

    for (i = 0; i < num_points; i++) {
        x = (double)rand() / RAND_MAX;
        y = (double)rand() / RAND_MAX;
        distance_squared = x * x + y * y;
        if (distance_squared <= radius_squared) {
            point_inside_circle++;
        }
    }
    pi_estimate = 4.0 * point_inside_circle / num_points;
    printf("Estimated Pi: %f\n", pi_estimate);
    return 0;
}

```

Here we use the simulation of random dropping points in the range of [0, 1] to estimate the value of pi.

MCTS is quite good to use in the information hidden environment.

```c
#include <stdio.h>
#include <stdlib.h>
#include <math.h>
#include <time.h>

// Possible actions in the game
#define ROCK 0 // Rock
#define SCISSORS 1 // Scissors
#define PAPER 2 // Paper
#define NUM_ACTIONS 3

// Probability of action of biased opponent
#define ROCK_PROB 50 // 50% of rock
#define SCISSORS_PROB 30 // 30% of scissors
#define PAPER_PROB 20 // 20% of paper

// Node structure
typedef struct {
int visits;
double wins;
double ucb;
} Node;

// Get opponent action based on probability distribution
int get_biased_opponent_action() {
int rand_num = rand() % 100;

if (rand_num < ROCK_PROB) {
return ROCK;
} else if (rand_num < ROCK_PROB + SCISSORS_PROB) {
return SCISSORS;
} else {
return PAPER;
}
}

// Use MCTS to select the best action and return win rate statistics
int mcts_best_action(int simulations, double win_rates[]) {
Node actions[NUM_ACTIONS] = {0};
int total_visits = 0;
int action_counts[NUM_ACTIONS] = {0};

// Main MCTS loop
for (int sim = 0; sim < simulations; sim++) {
// 1. Selection phase - using UCB formula
int selected_action = -1;
double best_ucb = -1.0;

for (int i = 0; i < NUM_ACTIONS; i++) {
if (actions[i].visits == 0) {
selected_action = i;
break;
}

// UCB calculation (exploration and exploitation balance)
actions[i].ucb = (actions[i].wins / actions[i].visits) +
sqrt(2.0 * log(total_visits) / actions[i].visits);

if (actions[i].ucb > best_ucb) {
best_ucb = actions[i].ucb;
selected_action = i;
}
}

// 2. Extension and 3. Simulation - guess opponent actions and simulate in an incomplete information environment
int opponent_action = get_biased_opponent_action();
action_counts[opponent_action]++;

// Calculation result: 0 = draw, 1 = win, -1 = lose
int result = 0;
if ((selected_action == ROCK && opponent_action == SCISSORS) ||
(selected_action == SCISSORS && opponent_action == PAPER) ||
(selected_action == PAPER && opponent_action == ROCK)) {
result = 1; // Victory
} else if (selected_action != opponent_action) {
result = -1; // Failure
}

// 4. Backpropagation
actions[selected_action].visits++;
actions[selected_action].wins += (result + 1) / 2.0; // Convert to [0,1] range
total_visits++;
}

// Calculate the winning rate of each action
for (int i = 0; i < NUM_ACTIONS; i++) {
if (actions[i].visits > 0) {
win_rates[i] = actions[i].wins / actions[i].visits;
} else {
win_rates[i] = 0.0;
}
}

// Print opponent action statistics
printf("Opponent action statistics (based on %d simulations):\n", simulations);
printf("- Rock: %.1f%%\n", (float)action_counts[ROCK] / simulations * 100);
printf("- Scissors: %.1f%%\n", (float)action_counts[SCISSORS] / simulations * 100);
printf("- Paper: %.1f%%\n", (float)action_counts[PAPER] / simulations * 100);
printf("\n");

// Select the action with the highest win rate
int best_action = 0;
double max_win_rate = win_rates[0];

for (int i = 1; i < NUM_ACTIONS; i++) {
if (win_rates[i] > max_win_rate) {
max_win_rate = win_rates[i];
best_action = i;
}
}

return best_action;
}

// Win rate evaluation name
const char* get_win_rate_evaluation(double rate) {
if (rate >= 0.7) return "excellent";
if (rate >= 0.6) return "very good";
if (rate >= 0.5) return "favorable";
if (rate >= 0.4) return "unfavorable";
if (rate >= 0.3) return "very bad";
return "very bad";
}

int main() {
const char* action_names[] = {"rock", "scissors", "cloth"};
srand(time(NULL));

printf("MCTS against preference opponent (50%%rock, 30%%scissors, 20%%cloth)\n");
printf("==================================================\n");

// Store win rate statistics
double win_rates[NUM_ACTIONS] = {0};

// Use MCTS to select actions and obtain win rate statistics
int best_action = mcts_best_action(100000, win_rates);

// Print win rate statistics table
printf("MCTS win rate statistics:\n");
printf("+--------+----------+-----------+-------------+\n");
printf("| Action | Win rate | Expected value | Evaluation |\n");
printf("+--------+----------+-----------+-------------+\n");

for (int i = 0; i < NUM_ACTIONS; i++) {
// Calculate expected value: win = 1, draw = 0, loss = -1 Convert back
double expected_value = (win_rates[i] * 2) - 1;

printf("| %-6s | %.2f%% | %+.2f | %-12s |\n",
action_names[i],
win_rates[i] * 100,
expected_value,
get_win_rate_evaluation(win_rates[i]));
}

printf("+--------+----------+-----------+-------------+\n");
printf("\nBest strategy: %s (win rate: %.2f%%)\n",
action_names[best_action], win_rates[best_action] * 100);

// Print theoretical optimal solution and compare with MCTS results
printf("\nTheoretical optimal strategy analysis:\n");
printf("- Rock: Expected value = 0.5×0(draw) + 0.3×1(win) + 0.2×(-1)(loss) = +0.10\n");
printf("- Scissors: Expected value = 0.5×(-1)(loss) + 0.3×0(draw) + 0.2×1(win) = -0.30\n");
printf("- Paper: Expected value = 0.5×1(win) + 0.3×(-1)(loss) + 0.2×0(draw) = +0.20\n");

return 0;
}
```

This is a rock-scissors-paper game. AI is hidden to be biased.

But the MCTS could find the best strategy to win the game.

Despite sometime writing a simulation code is not easy, but it is a good way to understand the algorithm.

