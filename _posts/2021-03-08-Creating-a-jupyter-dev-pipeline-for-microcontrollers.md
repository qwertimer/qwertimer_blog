---
toc: true
layout: post
description: Configuring a Development Environment for microcontrollers in Jupyter
categories: [Jupyter, Microcontrollers, Micropython, CircuitPython]
title: Micropython in Jupyter Notebooks
---


# Building a Development environment in jupyter for microcontrollers. 

The goal of this development was to build a pipeline to implement nbdev based code development with circuitpython. The initial issue that i faced was that the circuitpython jupyter kernel captured every message sent to jupyter and forwarded this over serial to the mcu. I had to capture this data before it was sent to be able to use it in python. I didn't want to effect the overall functionality of the circuitptyhon kernel so i chose to replicate the %python and %%python magics with one key difference. These magics did not create a subprocess by calling a command line based python. These parsed the python code to eval and exec commands. Below is the code written to implement the line magic %python. Currently the system has two flaws that need to be fixed the first of these is that the output of the commands implemented into the jupyter notebook do not display to the system. The second thing of note is that the system does not include a global scope for this information and as such the information isn't passed between cells. Further work aims to undertake this and help to catch some of the other errors that occur when running the code. 

```
 def is_magic(self, line):
        """Returns true if line was handled"""
        if line.startswith("%softreset"):
            self.board.softreset()
        elif line.startswith("%upload_delay"):
            try:
                s_line = line.split(' ')
                self.upload_delay = float(s_line[1])
                KERNEL_LOGGER.debug(f"upload_delay set to {float(s_line[1])} s")
            except TypeError:
                pass
        elif line.startswith("%python"):
            #python line magic, runs what ever is on the line following the %python magic.
            code = line.lstrip("%python")
            code = code.lstrip(' ')
            for item in code.split(";"):
                item = item.lstrip(' ') #remove leading white space
                try:
                    print(eval(item))   #does not print
                except:
                    out = exec(item)
                    if out != None:
                        print(out)      #does not print
            
        else:
            return False
        return True
```


The cell magics code is shown below. If you wish to play with this library feel free to install my fork of the adafruit circuitpython jupyter kernel from this link (https://github.com/qwertimer/circuitpython_jupyter_kernel) or if my pull request is accepted the latest versions of the adafruit library.

```

 def is_cell_magic(self, code):
        """Cell magic to run python code.
        -----
        Cell shall begin with %%python followed by a new line
        Will iteratively run each line of code.
        """

        if code.startswith("%%python"):
            code = code.lstrip("%%python")
            code = code.lstrip(' ')
            data = code.splitlines(True)
            for item in data:
                
                code = code.lstrip(' ')    #this removes all preceeding white space, 
                                           #i need to figure out how to get for loops, etc working
                try:
                    print(eval(item))      #does not print
                except:
                    out = exec(item)

                    if out != None:
                        print(out)         #does not print
            return True
        else:
            return False
```

Finally, this is a work in progress and further magics may be implemented to run makefiles and other CI based tests in the jupyter notebook. One day this system will be a fully fledged repl and development environment for circuitpython.


