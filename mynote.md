from tkinter import *
import time
import tkinter.filedialog
import tkinter.colorchooser
import tkinter.messagebox
import tkinter.scrolledtext
import tkinter.ttk
import tkinter.simpledialog
import requests

class Watch(Frame):
    msec = 1000
    def __init__(self, parent=None, **kw):
            Frame.__init__(self, parent, kw)
            self._running = False
            self.timestr1 = StringVar()
            self.timestr2 = StringVar()
            self.makeWidgets()
            self.flag  = True
    def makeWidgets(self):
        l1 = Label(self, textvariable = self.timestr1,font=('Arial',14))
        l2 = Label(self, textvariable = self.timestr2,font=('Arial',14))
        l1.pack()
        l2.pack()
    def _update(self):
        self._settime()
        self.timer = self.after(self.msec, self._update)
    def _settime(self):
        today1 = str(time.strftime('%Y-%m-%d', time.localtime(time.time())))
        time1 = str(time.strftime('%H:%M:%S', time.localtime(time.time())))
        self.timestr1.set(today1)
        self.timestr2.set(time1)
    def start(self):
        self._update()
        self.pack()

def Start():
    window = tkinter.Tk()
    window.title('My Note')
    window['width']=1000
    window['height']=800
    textChanged = tkinter.IntVar()
    #当前文件名
    filename =''
    #创建菜单
    menu = tkinter.Menu(window)

    def Open():
        global filename
        #如果内容已改变，先保存
        if textChanged.get():
            yesno = tkinter.messagebox.askyesno(title='保存么?',message='要保存文件吗?')
            if yesno == tkinter.YES:
                Save()
        filename = tkinter.filedialog.askopenfilename(title='打开文件',filetypes=[('Text files','*.txt')])
        if filename:
        #清空内容,0.0是lineNumber.Column的表示方法
            txtContent.delete(0.0, tkinter.END)
            with open(filename,'r') as fp:
                txtContent.insert(tkinter.INSERT,''.join(fp.readlines()))
            fp.close()
            #标记为尚无修改
            textChanged.set(0)

    def Save():
        global filename
        #如果是第一次保存新建文件，则打开“另存为”窗口
        if not filename:
            SaveAs()
        #如果内容发生改变，保存
        elif textChanged.get():
            with open(filename,'r') as fp:
                fp.write(txtContent.get(0.0,tkinter.END))
            fp.close()
        textChanged.set(0)

    def SaveAs():
        global filename
    #打开“另存为”窗口
        newfilename = tkinter.filedialog.asksaveasfilename(title='另存为...',initialdir=r'c:\\',initialfile='新文件.txt')
    #如果指定了文件名，则保存文件
        if newfilename:
            fp = open(newfilename,'w')
            fp.write(txtContent.get(0.0, tkinter.END))
            fp.close()
            filename = newfilename
        textChanged.set(0)

    def Close():
        global filename
        Save()
        txtContent.delete(0.0, tkinter.END)
    #置空文件名
        filename =''

    def Undo():
    #启用undo标志
        txtContent['undo']=True
        try:
            txtContent.edit_undo()
        except Exception as e:
            pass

    def Redo():
        txtContent['undo']=True
        try:
            txtContent.edit_redo()
        except Exception as e:
            pass

    def Copy():
        txtContent.clipboard_clear()
        txtContent.clipboard_append(txtContent.selection_get())

    def Cut():
        Copy()
    #删除所选内容
        txtContent.delete(tkinter.SEL_FIRST, tkinter.SEL_LAST)

    def Paste():
    #如果没有选中内容，则直接粘贴到鼠标位置
    #如果有所选内容，则先删除再粘贴
        try:
            txtContent.insert(tkinter.SEL_FIRST, txtContent.clipboard_get())
            txtContent.delete(tkinter.SEL_FIRST, tkinter.SEL_LAST)
    #如果粘贴成功就结束本函数，以免异常处理结构执行完成之后再次粘贴
            return
        except Exception as e:
            pass
            txtContent.insert(tkinter.INSERT, txtContent.clipboard_get())

    def Search():
    #获取要查找的内容
        textToSearch = tkinter.simpledialog.askstring(title='Search',prompt='What to search?')
        start = txtContent.search(textToSearch,0.0, tkinter.END)
        if start:
            tkinter.messagebox.showinfo(title='Found', message='Ok')
        else:
            tkinter.messagebox.showerror(title='Not Found',message='Fail')

    def About():
        tkinter.messagebox.showinfo(title='关于', message='并没有帮助')
            
    #File菜单
    submenu = tkinter.Menu(menu, tearoff=0)
    submenu.add_command(label='打开', command=Open)
    submenu.add_command(label='保存', command=Save)
    submenu.add_command(label='另存为', command=SaveAs)
    submenu.add_separator()
    submenu.add_command(label='关闭', command=Close)
    menu.add_cascade(label='文件', menu=submenu)
    #Edit菜单
    submenu = tkinter.Menu(menu, tearoff=0)
    submenu.add_command(label='Undo', command=Undo)
    submenu.add_command(label='Redo', command=Redo)
    submenu.add_separator()
    submenu.add_command(label='拷贝', command=Copy)
    submenu.add_command(label='剪切', command=Cut)
    submenu.add_command(label='粘贴', command=Paste)
    submenu.add_separator()
    submenu.add_command(label='查找', command=Search)
    menu.add_cascade(label='编辑', menu=submenu)
    #Help菜单
    submenu = tkinter.Menu(menu, tearoff=0)
    submenu.add_command(label='关于', command=About)
    menu.add_cascade(label='帮助', menu=submenu)

    #将创建的菜单关联到应用程序窗口
    window.config(menu=menu)
    #创建文本编辑组件
    txtContent = tkinter.scrolledtext.ScrolledText(window, wrap=tkinter.WORD)
    txtContent.pack(fill=tkinter.BOTH, expand=tkinter.YES)

    def KeyPress(event):
        textChanged.set(1)
    txtContent.bind('<KeyPress>',KeyPress)

    window.mainloop()

def main():
    app=Tk()
    app.title('Frame')
    

    frame1=Frame(app,width=50,height=300).pack(side='right')

    canvas1=Canvas(app,bg='red',width=500,height=300)
    imag1=PhotoImage(file='1234567.png')
    canvas1.create_image(0,175,image=imag1)
    canvas1.pack(side='left')

    mw = Watch(app)
    mw.start()
    #获取天气
    url="http://www.tianqi.com/wuhan/"
    r=requests.get(url)
    kv={'user-agent':'Mozilla/5.0'}
    r=requests.get(url,headers=kv)
    from bs4 import BeautifulSoup
    demo=r.text[9000:10000]
    soup=BeautifulSoup(demo,'html.parser')
    Label(frame1,text=soup.span.b.string,font=('Arial',14)).pack()
    Label(frame1,text=soup.span.b.next_sibling.string,font=('Arial',14)).pack()
    
    kaishi=Button(frame1,text='start',command=Start).pack()
    app.mainloop()
        

if __name__=='__main__':
    main()
