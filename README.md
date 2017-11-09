Nochi --- Minimalistic AlphaGoZero-like Engine
==============================================

Nochi is a fork of the Michi minimalistic Computer Go engine that replaces
Monte Carlo simulations with a Keras neural model.  It is an attempt to
reproduce the AlphaGo Zero work on a small scale.

It is by no means as polished as Michi, but might still be useful as an
inspiration.

This is a truly "zero-knowledge" system like AlphaGo zero, but it's
not entirely 1:1, I did some tweaks which I thought might help early
convergence:

  * AlphaGo used 19 resnet layers for 19x19, so I used 7 layers.
  * The neural network is updated after _every_ game, _twice_, on _all_
    positions plus 64 randomly sampled positions from entire history,
    this all done four times - on original position and various
    symmetry flips (but I was too lazy to implement 90\deg rotation).
  * Instead of supplying last 8 positions on input of the network,
    I feed just the last position plus two indicator matrices showing
    the location of the last and second-to-last move.
  * No symmetry pruning during tree search.
  * Value function is trained with cross-entropy rather than MSE.
  * No resign auto-threshold but it is important to play 25% games
    without resigning to escale local "optima".
  * 1/Temperature is 2 for first three moves.
  * It uses a different number of simulations per move.

It has been verified to be able to get near GNUGo level in ~2 weeks (8500
games, 6 threads + 1x Tesla M60) on 7x7.  Not terribly great but it clearly
does something.

Nochi is distributed under the MIT licence.  Now go forth, hack and peruse!

Usage
-----

First, decide on the board size and edit the N parameter at the script
beginning.  N=19 is the default but you may want to use e.g. N=7.

To start training,

	python ./michi.py selfplay

which will autogenerate a run id and make periodical snapshot (the numbers
should be multiplied by thread count to get true number of games played).  To
resume in any mode (e.g. even start gameio or gtp), add the id as a parameter:

	python ./michi.py selfplay G171107T013304_000000150

To play, you can e.g. pass the gtp argument and start it in gogui, or let it
play a bunch of games with GNUGo:

	gogui-1.4.9/bin/gogui-twogtp -black 'python ./michi.py gtp G171107T013304_000000150' -white 'gnugo --mode=gtp --chinese-rules --capture-all-dead' -size 7 -komi 7.5 -verbose -auto -alternate -games 20 -sgffile x

Nochi also supports supervised training:

	while true; do find GoGoD-2008-Winter-Database/ -name '*.sgf' | shuf; done | python ./michi.py replay_train

To mix value supervision with actual MCTS training signal for positional
output, use smt. like:

	while true; do find GoGoD-2008-Winter-Database/ -name '*.sgf' | shuf; done | python ./michi.py replay_traindist G171107T224743_R000030000

You can freely switch between selfplay, supervised and supervised+MCTS using
snapshots, they are compatible.
