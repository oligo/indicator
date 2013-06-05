#!/usr/bin/env python
""" 
This example illustrates embedding a visvis figure in a wx application.
"""

import wx
import visvis as vv
import time
import numpy as np
import threading
import Queue
# import third-party module
import psutil

LIMIT=100
# Create a visvis app instance, which wraps a wx application object.
# This needs to be done *before* instantiating the main window. 
app = vv.use('wx')
queue = Queue.Queue()
cpudata = list()
vmemdata = list()
smemdata = list()

class MainWindow(wx.Frame):
    def __init__(self):
        wx.Frame.__init__(self, None, -1, "Embedding in WX", size=(560, 420))
        

        # Make a panel with a button
        self.leftpanel = wx.Panel(self)
        but = wx.Button(self.leftpanel, -1, 'Click me')
        self.status = wx.StaticText(self.leftpanel, label="CPU:\nMemory:", style=wx.ALIGN_LEFT)
        panelSizer = wx.BoxSizer(wx.VERTICAL)
        panelSizer.Add(but, 0, wx.LEFT, 10)
        panelSizer.Add(self.status, 0, wx.LEFT, 10)
        self.leftpanel.SetSizer(panelSizer)
        

        # Make figure using "self" as a parent
        Figure = app.GetFigureClass()
        self.fig = Figure(self)
        self.fig.enableUserInteraction = False
        self.timer = vv.Timer(self.fig, interval = 1000, oneshot=False)
        self.timer.Bind(self._Plot)
        
        # Make sizer and embed stuff
        self.sizer = wx.BoxSizer(wx.HORIZONTAL)
        self.sizer.Add(self.leftpanel, 0, wx.EXPAND,20)
        self.sizer.Add(self.fig._widget, 3, wx.EXPAND,20)
        
        # Make callback
        #but.Bind(wx.EVT_BUTTON, self._Plot)
        
        # Apply sizers        
        self.SetSizer(self.sizer)
        self.SetAutoLayout(True)
        self.Layout()   
        

        # Finish
        self.Show()


    def plotOnTimer(self):
        self.timer.Start()
    
    def _Plot(self, event):
        
        # update status text
        self.status.SetLabel("CPU:%.1f\nMemory:%.1f" % (cpudata[-1], vmemdata[-1]))
        # Make sure our figure is the active one
        # If only one figure, this is not necessary.
        #vv.figure(self.fig.nr)
        
        # Clear it
        vv.clf()
        
        # Plot
        a = vv.subplot(211)
        vv.plot(cpudata, lw=2, lc="c", mc ='y', ms='d')
        vv.legend("cpu percent ")
        a.axis.showGrid = True
        a.axis.xlabel = "CPU Usage Percent"
        a.axis.ylabel = "sampling time line"

        a = vv.subplot(212)
        vv.plot(vmemdata, lw=2, mew=1, lc='m', mc ='y', ms='d' )
        vv.legend("virtual memory")
        a.axis.showGrid = True
        #a.axis.xlabel = "Memory Usage Percent"
        #a.axis.ylabel = "sampling time line"

        #vv.plot([1,2,3,1,6])
        #vv.hist([1,2,3,1,6])
    
#----------------------------
# Data source
#----------------------------

class Indicator(threading.Thread):
    ''' An system resource indicator'''
    def __init__(self, queue):
        super(Indicator, self).__init__()
        self.queue = queue

    def cpu(self):
        cpuPercent = psutil.cpu_percent(interval=1, percpu=False)
        self.toQueue(('cpu', cpuPercent))
        #if type(cpuPercent) == list:
        #    self.toQueue(('cpu', cpuPercent))

    def vmem(self):
        '''Virtual Memory percent'''
        memUsage = psutil.virtual_memory()
        self.toQueue(('vmem',memUsage.percent))

    def smem(self):
        '''Swap Memory percent'''
        smem = psutil.swap_memory()
        #self.toQueue(('smem', [smem.total, smem.used, smem.percent]))
        self.toQueue(('smem', smem.percent))

    def toQueue(self, data):
        self.queue.put(data)

    def run(self):
        while True:
            self.cpu()
            self.vmem()
            self.smem()
            #self.queue.task_done()

class ThreadPlot(threading.Thread):
    ''' plot thread'''
    def __init__(self, queue):
        super(ThreadPlot, self).__init__()
        self.queue = queue

    def run(self):
        global cpudata 
        global vmemdata
        global smemdata
        while True:
            data = self.queue.get()
            if data[0] == 'cpu':
                cpudata.append(data[1])
                if len(cpudata) > LIMIT:
                    cpudata = cpudata[(len(cpudata)-LIMIT):]
            elif data[0] == 'vmem':
                vmemdata.append(data[1])
                if len(vmemdata) > LIMIT:
                    vmemdata = vmemdata[(len(vmemdata)-LIMIT):]
            elif data[0] == 'smem':
                smemdata.append(data[1])
                if len(vmemdata) > LIMIT:
                    smemdata = smemdata[(len(smemdata)-LIMIT):]
            
            self.queue.task_done()


            

def indicator():
    indicatorThread = Indicator(queue)
    indicatorThread.setDaemon(True)
    indicatorThread.start()

    # start plot data gathering thread
    t = ThreadPlot(queue)
    t.setDaemon(True)
    t.start()


if __name__ == '__main__':

    indicator()
    # Two ways to create the application and start the main loop
    if False:
        # The visvis way. Will run in interactive mode when used in IEP or IPython.
        app.Create()
        m = MainWindow()
        app.Run()
    
    else:
        # The native way.
        wxApp = wx.App()    
        m = MainWindow()
        m.plotOnTimer()
        wxApp.MainLoop()
