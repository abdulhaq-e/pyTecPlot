import re
import numpy as np
from scipy.interpolate import griddata
import matplotlib.pyplot as plt
from itertools import islice
import os


def Read(DataFile):

        #Getting the header of the files in order to get number of points and cells
        with open(DataFile) as f:
                head=list(islice(f, 3)) #reading the first 3 lines
        #finding n and e using regular expressions, we expect both to be in the third line
        n = re.findall(r'n= +(\d*)', str(head)) # no. of nodes
        e = re.findall(r'e= +(\d*)', str(head)) #no. of cells
        
        #just in case things went wrong:
        if (n==[] or e==[]) is True:
                print "I couldn't read the number of cells or the number of cells or both, here are a few thoughts on why: \n 1- They're not located on line 3 \n 2- There's a space after between n or e and the equal sign."
                return
        else:


                #if everything is fine, keep going:
                n = int(n[0])
                e = int(e[0])
                x_y_lines = (n/20) + ( (n % 20)!=0 ) #no. of lines for x and y coordinates, the last bit is the remainder.
                var_lines = (e/20) + ( (e % 20)!=0 ) #no. of lines for the variables, whatever they may be.
                nodes_lines = e/4

                #let's separate stuff now:
                start_x = 3
                end_x = x_y_lines + start_x

                start_y = end_x
                end_y = start_y + x_y_lines

                start_omega = end_y
                end_omega = start_omega + var_lines

                start_nodes = end_y + (var_lines*5) + 1
                end_nodes = start_nodes + e

		coordinates_line = end_nodes + 2
		start_aerofoil = end_nodes + 3
                with open(DataFile) as f:
                        x_mixed = list(islice(f, start_x, end_x))
                with open(DataFile) as f:
                        y_mixed = list(islice(f, start_y, end_y))
                        #i'm only going to get the mag of vorticity, other variables can be easily added
                with open(DataFile) as f:
                        omega_mixed = list(islice(f, start_omega, end_omega))
                #put the other variables IF wanted here
                with open(DataFile) as f:
                        nodes_mixed = list(islice(f,start_nodes, end_nodes))

		#Good idea to get the aerofoil coordinates an plot them as well
                with open(DataFile) as f:
                        coordinates_size = list(islice(f,coordinates_line, coordinates_line+1))
                coordinates_size = np.array((re.findall('\d+',str(coordinates_size))), dtype=np.int)
		end_aerofoil = start_aerofoil + coordinates_size
                with open(DataFile) as f:
                        af_coord = list(islice(f,start_aerofoil, end_aerofoil))

                #now the real fun starts, regular expression!
                x_nodes = np.array(re.findall(r'-?\d+\.?\d*', str(x_mixed)), dtype=np.float64)
                y_nodes = np.array(re.findall(r'-?\d+\.?\d*', str(y_mixed)), dtype=np.float64)
                omega_centred = np.array(re.findall(r'-?\d+\.?\d*',str(omega_mixed)), dtype=np.float64)
                nodes = np.array(re.findall(r'\d+',str(nodes_mixed)), dtype=np.int)
                nodes = nodes.reshape(e,4)

                #let's find the center of each cell
                nodes = nodes[:] - 1
                x_elements = x_nodes[nodes]
                y_elements = y_nodes[nodes]

                x_centred = (x_elements[:][:,0]+ x_elements[:][:,1])/2
                y_centred = (y_elements[:][:,0]+ y_elements[:][:,1])/2
                
		#we eventually have to use our own data
                x_min = x_centred.min()
                x_max = x_centred.max()

                y_min = y_centred.min()
                y_max = y_centred.max()

                #the interested parts seem to be here
                x_fancy1 = 5 
                x_fancy2 = 10
                y_fancy1 = 5
                y_fancy2 = 10


                #almost there, new data
                k = 5000
                X1 = np.linspace(x_min, x_fancy1, 0.2*k)
		X2 = np.linspace(x_fancy1, x_fancy2, 0.7*k)
                X3 = np.linspace(x_fancy2, x_max, 0.1*k)
                X = X1
                X = np.append(X, X2)
                X = np.append(X, X3)
                
                Y1 = np.linspace(y_min, y_fancy1, 0.2*k)
                Y2 = np.linspace(y_fancy1, y_fancy2, 0.7*k)
                Y3 = np.linspace(y_fancy2, y_max, 0.1*k)
                Y = Y1
                Y = np.append(Y, Y2)
                Y = np.append(Y, Y3)


                XX, YY = np.meshgrid(X, Y)
                OMEGA = griddata((x_centred, y_centred), omega_centred, (XX,YY), method='nearest')

                #aerofoil
                aerofoil = np.array(re.findall(r'-?\d+\.?\d*',str(af_coord)), dtype=np.float64)
                aerofoil = aerofoil.reshape(coordinates_size[0],2)
		#finally
                plt.contourf(XX, YY, OMEGA)
                plt.plot(aerofoil[:,0], aerofoil[:,1], lw=3, c='k')
                plt.show()

