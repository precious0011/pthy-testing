Exception in Tkinter callback
Traceback (most recent call last):
  File "C:\Program Files\Python312\Lib\tkinter\__init__.py", line 1967, in __call__
    return self.func(*args)
           ^^^^^^^^^^^^^^^^
  File "\\dom.sandwell.ac.uk\Staff\Home\Cadbury & Sandwell Shared Folders\T-Level Digital OS Exam Files\tlevel-digital-os-11\Version control\Version 4.py", line 114, in login
    open_customer_dashboard(user)
  File "\\dom.sandwell.ac.uk\Staff\Home\Cadbury & Sandwell Shared Folders\T-Level Digital OS Exam Files\tlevel-digital-os-11\Version control\Version 4.py", line 303, in open_customer_dashboard
    lbl = tk.Label(box, image=img)
          ^^^^^^^^^^^^^^^^^^^^^^^^
  File "C:\Program Files\Python312\Lib\tkinter\__init__.py", line 3237, in __init__
    Widget.__init__(self, master, 'label', cnf, kw)
  File "C:\Program Files\Python312\Lib\tkinter\__init__.py", line 2648, in __init__
    self.tk.call(
_tkinter.TclError: image "25" doesn't exist
