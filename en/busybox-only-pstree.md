title=Busybox-only pstree
intro=For when you're on limited environment but want to see a process tree
tags=bash
created=2020-01-31

Just use this script:

	#!/bin/busybox ash
	#
	# Script to show process tree.
	# Accepts one optional argument:
	# pid of root process - by default 1 (init)

	cd /tmp

	# Build list of 'parent child' pids
	grep 'PPid:' /proc/*/status | sed -r 's_/proc/([^/]*)/[^0-9]*([0-9]*)_\2 \1_' >ppids.ps

	# Preserve ps output to show
	ps w >ps.ps

	# own pid - we don't show own children because they're guaranteed
	# to change between above two calls
	self=$$

	# function to show one process with all children, recursively
	# args:
	# * pid of process to show
	# * indent - string of spaces to print before current line
	showproc()
	{
		# print current process
		echo "$2$(grep "^ *$1 " ps.ps || echo "$1 ???")"
		# don't print own children
		test $1 == $self && return
		# print children, adding two spaces to indent
		for subpid in `grep "^$1 " ppids.ps | awk '{print $2}'`; do
			showproc $subpid "  $2"
		done
	}

	# start with root process (pid 1 by default)
	showproc ${1:-1} ''

<script src="/microlight.js"></script>
