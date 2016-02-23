#include "mainwindow.h"
#include "ui_mainwindow.h"

MainWindow::MainWindow(QWidget *parent) :
    QMainWindow(parent),
    ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    numFoto=0;
    cap = new VideoCapture(0);
    if(!cap->isOpened())
        cap = new VideoCapture(1);
    capture = true;
    showColorImage = false;
    winSelected = false;
    cap->set(CV_CAP_PROP_FRAME_WIDTH, 320);
    cap->set(CV_CAP_PROP_FRAME_HEIGHT, 240);
    imgS = new QImage(320,240, QImage::Format_RGB888);
    visorS = new RCDraw(320,240, imgS, ui->imageFrameS);
    imgD = new QImage(320,240, QImage::Format_RGB888);
    visorD = new RCDraw(320,240, imgD, ui->imageFrameD);



    colorImage.create(240,320,CV_8UC3);
    grayImage.create(240,320,CV_8UC1);
    destColorImage.create(240,320,CV_8UC3);
    Black_Color_Image.create(240,320,CV_8UC3);
    Black_Gray_Image.create(240,320,CV_8UC1);

    destGrayImage.create(240,320,CV_8UC1);
    gray2ColorImage.create(240,320,CV_8UC3);
    destGray2ColorImage.create(240,320,CV_8UC3);

    connect(&timer,SIGNAL(timeout()),this,SLOT(compute()));
    connect(ui->captureButton,SIGNAL(clicked(bool)),this,SLOT(start_stop_capture(bool)));
    connect(ui->colorButton,SIGNAL(clicked(bool)),this,SLOT(change_color_gray(bool)));
    connect(ui->ImageButton,SIGNAL(clicked()),this,SLOT(set_Image()));
    connect(ui->SaveButton,SIGNAL(clicked()),this,SLOT(save_Image()));
    connect(ui->LoadButton,SIGNAL(clicked()),this,SLOT(load_Image()));
    connect(ui->ResizeButton,SIGNAL(clicked()),this,SLOT(resize_Image()));
    connect(ui->EnlargeButton,SIGNAL(clicked()),this,SLOT(enlarge_Image()));
    connect(ui->WarpButton,SIGNAL(clicked(bool)),this,SLOT(warped(bool)));

    connect(visorS,SIGNAL(windowSelected(QPointF, int, int)),this,SLOT(selectWindow(QPointF, int, int)));
    connect(visorS,SIGNAL(pressEvent()),this,SLOT(deselectWindow()));
    timer.start(60);


}

MainWindow::~MainWindow()
{
    delete ui;
    delete cap;
    delete visorS;
    delete visorD;
    delete imgS;
    delete imgD;

}

void MainWindow::compute()
{

    if(capture && cap->isOpened())
    {
        *cap >> colorImage;

        cvtColor(colorImage, grayImage, CV_BGR2GRAY);
        cvtColor(colorImage, colorImage, CV_BGR2RGB);

    }

    if(warpeded){
        //warpeded=false;
        rotate_Image();
    }


    if(showColorImage)
    {
        memcpy(imgS->bits(), colorImage.data , 320*240*3*sizeof(uchar));
        memcpy(imgD->bits(), destColorImage.data , 320*240*3*sizeof(uchar));
    }
    else
    {
        cvtColor(grayImage,gray2ColorImage, CV_GRAY2RGB);
        cvtColor(destGrayImage,destGray2ColorImage, CV_GRAY2RGB);
        memcpy(imgS->bits(), gray2ColorImage.data , 320*240*3*sizeof(uchar)); //Pasa el contenido a VisorS
        memcpy(imgD->bits(), destGray2ColorImage.data , 320*240*3*sizeof(uchar));//Pasa el contenido a VisorD

    }


    if(winSelected)
    {
        visorS->drawSquare(QPointF(imageWindow.x+imageWindow.width/2, imageWindow.y+imageWindow.height/2), imageWindow.width,imageWindow.height, Qt::green );

    }
    visorS->update();
    visorD->update();




}
void MainWindow::start_stop_capture(bool start)
{
    if(start)
    {
        ui->captureButton->setText("Pulsar para Parar");
        capture=true;
            }
    else
    {
        ui->captureButton->setText("Pulsar Para Capturar");
        capture = false;
    }
}
void MainWindow::set_Image()
{
 if(winSelected){
    // ui->textEdit->setText("Hola");
//colorImage.copyTo();

   //  destColorImage.resize(colorImage.rowRange(imageWindow.x,imageWindow.width).colRange(imageWindow.y,imageWindow.height));


    int centro_wdt_image=imageWindow.width/2;
    int centro_hgt_image=imageWindow.height/2;
    //ui->textEdit->setText(centro_hgt_image);
    printf("Altura : %d\nn",centro_hgt_image);

    int origen_x =160-centro_wdt_image;
    int origen_y =120-centro_hgt_image;


    int fin_x =origen_x+imageWindow.width;
    int fin_y =origen_y+imageWindow.height;


    destColorImage=Black_Color_Image.clone();
    destGrayImage=Black_Gray_Image.clone();

if(showColorImage){
    Mat cuadroImagen=colorImage.colRange(imageWindow.x,imageWindow.x+imageWindow.width).rowRange(imageWindow.y,imageWindow.y+imageWindow.height);

    cuadroImagen.copyTo(destColorImage.colRange(origen_x,fin_x).rowRange(origen_y,fin_y));
}
else{
    Mat cuadroImagen=grayImage.colRange(imageWindow.x,imageWindow.x+imageWindow.width).rowRange(imageWindow.y,imageWindow.y+imageWindow.height);

        cuadroImagen.copyTo(destGrayImage.colRange(origen_x,fin_x).rowRange(origen_y,fin_y));
}
   // cuadroImagen.copyTo(destGrayImage.colRange(origen_x,fin_x).rowRange(origen_y,fin_y));
 }

 else{
     destColorImage=colorImage.clone();
     destGrayImage=grayImage.clone();
 }
}
void MainWindow::rotate_Image(){
int numeroHorizontal=ui->HorizontalTranslation->value();
float angle=ui->Dial->value()/10.;
int numerovertical=ui->VerticalTranslation->value();
Mat MT(2,3,CV_32FC1);

destColorImage.setTo(Scalar(0,0,0));//=Black_Color_Image.clone();
destGrayImage.setTo(0);//=Black_Gray_Image.clone();

MT.row(0).col(0)=cos(angle);
MT.row(0).col(1)=sin(angle);
MT.row(0).col(2)=numeroHorizontal;

MT.row(1).col(0)=-sin(angle);
MT.row(1).col(1)=cos(angle);
MT.row(1).col(2)=numerovertical;
Mat Aux;
Mat Aux_Grey;

Rect MarcoCentral;
MarcoCentral.width=320;
MarcoCentral.height=240;


cv::resize(colorImage,Aux,Size(0,0),ui->Zoom->value(),ui->Zoom->value());
cv::resize(grayImage,Aux_Grey,Size(0,0),ui->Zoom->value(),ui->Zoom->value());

MarcoCentral.x=Aux.cols/2-160;
MarcoCentral.y=Aux.rows/2-120;

warpAffine(Aux(MarcoCentral), destColorImage, MT, Size(320,240));
warpAffine(Aux_Grey(MarcoCentral), destGrayImage, MT, Size(320,240));

}
void MainWindow::save_Image()
{
 savebool=true;
bool test=capture;
capture=false;
 //ui->textEdit->setText("Save Image");

 QString filename=QFileDialog::getSaveFileName(0,"Save file",QDir::currentPath(),
"JPG files (*.jpg);;BMP files (*.bmp);;PNG files (*.png);;All files (*.*)",
     new QString("Text files (*.txt)"));

 if(!filename.isEmpty()){

 if(showColorImage){
    cvtColor(colorImage, colorImage, CV_BGR2RGB); //
    imwrite(filename.toStdString(), colorImage);
 }
 else{
    imwrite(filename.toStdString(), grayImage);
 }
}
capture=test;

}
void MainWindow::load_Image()
{

loadbool=true;
//ui->textEdit->setText("Load Image");

QString filters("JPG files (*.jpg);;BMP files (*.bmp);;PNG files (*.png);;All files (*.*)");
QString defaultFilter("All files (*.*)");

QFileDialog fileDialog(0, "Open file", "/home/guille/Escritorio", filters);
fileDialog.selectNameFilter(defaultFilter);
QStringList fileNames;
if (fileDialog.exec())
    fileNames = fileDialog.selectedFiles();

QString File=fileNames.front();
fileNames.pop_front();
colorImage=imread(File.toStdString());

cv::resize(colorImage,colorImage, Size(320,240),0,0,INTER_LINEAR);

cvtColor(colorImage, grayImage, CV_BGR2GRAY);
cvtColor(colorImage,colorImage, CV_BGR2RGB);

start_stop_capture(false);
}
void MainWindow::resize_Image()
{
 resizebool=true;

 if(winSelected){

    Mat cuadroImagen=colorImage.colRange(imageWindow.x,imageWindow.x+imageWindow.width).rowRange(imageWindow.y,imageWindow.y+imageWindow.height);
    cv::resize(cuadroImagen,cuadroImagen, Size(320,240),0,0,INTER_LINEAR);
    destColorImage=cuadroImagen.clone();

    Mat cuadroImagen_Gray=grayImage.colRange(imageWindow.x,imageWindow.x+imageWindow.width).rowRange(imageWindow.y,imageWindow.y+imageWindow.height);
    cv::resize(cuadroImagen_Gray,cuadroImagen_Gray, Size(320,240),0,0,INTER_LINEAR);
    destGrayImage=cuadroImagen_Gray.clone();
    }
}
void MainWindow::enlarge_Image()
{
 resizebool=true;

 if(winSelected){

      float fx=0.0,fy=0.0, f=0.0;

      float Height=imageWindow.height;
      float Width=imageWindow.width;


      fy=240./Height;
      fx=320./Width;

     destColorImage.setTo(Scalar(0,0,0));//=Black_Color_Image.clone();
     destGrayImage.setTo(0);//=Black_Gray_Image.clone();


     if(fx<fy) f = fx;
     else f = fy;

     Rect marcoFinal;
     marcoFinal.height  =rint( Height * f);
     marcoFinal.width = rint( Width * f);

     qDebug()<<destColorImage.size().width<<destColorImage.size().height;

     if(fx > fy){
         marcoFinal.x = (destColorImage.size().width / 2.0 - marcoFinal.width/ 2.0);
         marcoFinal.y = 0;
     }
     else if(fx < fy)
     {
         marcoFinal.x = 0;
         marcoFinal.y = (destColorImage.size().height / 2.0 - marcoFinal.height/ 2.0);
     }
     else{
         //ui->textEdit->setText("Enlarge !!");
     }

     qDebug()<<marcoFinal.x<<marcoFinal.y<<marcoFinal.width<<marcoFinal.height;
     cv::resize(colorImage(imageWindow),destColorImage(marcoFinal),Size(0,0),f,f);
     cv::resize(grayImage(imageWindow),destGrayImage(marcoFinal),Size(0,0),f,f);
}
}
void MainWindow::change_color_gray(bool color)
{
    if(color)
    {
        ui->colorButton->setText("Gray image");
        showColorImage = true;
    }
    else
    {
        ui->colorButton->setText("Color image");
        showColorImage = false;
    }
}
void MainWindow::selectWindow(QPointF p, int w, int h)
{
    QPointF pEnd;
    if(w>0 && h>0)
    {
        imageWindow.x = p.x()-w/2;
        if(imageWindow.x<0)
            imageWindow.x = 0;
        imageWindow.y = p.y()-h/2;
        if(imageWindow.y<0)
            imageWindow.y = 0;
        pEnd.setX(p.x()+w/2);
        if(pEnd.x()>=320)
            pEnd.setX(319);
        pEnd.setY(p.y()+h/2);
        if(pEnd.y()>=240)
            pEnd.setY(239);
        imageWindow.width = pEnd.x()-imageWindow.x;
        imageWindow.height = pEnd.y()-imageWindow.y;

        winSelected = true;


    }
}
void MainWindow::warped(bool clicked)
{
warpeded=clicked;
}
void MainWindow::deselectWindow()
{
    winSelected = false;
}


