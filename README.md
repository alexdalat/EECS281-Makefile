
### Original: 

https://github.com/eecs281staff/Makefile

### To be used with:

https://github.com/alexdalat/perfanno-vscode

### Relevant Addition:

```make
# CUSTOMIZE
PERF_MODE := profile
PERF_EXECUTE := $(EXECUTABLE)_$(PERF_MODE) -m SOME_MODE < sample-data.txt
PERF_DENSITY := 4000

# Escape special characters
PERF_EXECUTE_ESCAPED:=$(subst $\',$\'$\"$\'$\"$\',$(PERF_EXECUTE))
LOCAL_DIR := $(shell pwd)/
REMOTE_DIR := /home/$(UNIQNAME)/$(REMOTE_PATH)/
OS := $(shell uname -s)
perf_caen:
	# Step 1: Sync files to CAEN
	$(MAKE) sync2caen
	# Step 2: SSH to server, build, record performance, and generate report
	ssh $(UNIQNAME)@login.engin.umich.edu 'source /etc/profile && cd ~/$(REMOTE_PATH) && module load gcc/11.3.0 && make $(PERF_MODE) && rm perf.data &>/dev/null; perf record -F $(PERF_DENSITY) --call-graph dwarf ./$(PERF_EXECUTE_ESCAPED) > /dev/null && perf report -g folded,0,caller,srcline,branch,count --no-children --full-source-path --stdio -i perf.data > perf.out'
	# Step 3: Fetch the performance output file
	scp $(UNIQNAME)@login.engin.umich.edu:$(REMOTE_PATH)/perf.out .
	# Step 4: Replace remote path with local directory path in perf.out
ifeq ($(OS),Darwin)
	sed -i '' 's:$(REMOTE_DIR):$(LOCAL_DIR):g' perf.out
else
	sed -i 's:$(REMOTE_DIR):$(LOCAL_DIR):g' perf.out
endif
	@echo "Performance report saved to perf.out"
.PHONY: perf_caen
```

