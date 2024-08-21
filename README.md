# an-lisis-de-datos-Geofisicos

% Certamen 1 ADG 2022 - CRISTIAN MORENO MUÑOZ

%% 1.a y b) Se cargan los datos y se borran la primera y ultima fila manualmente
%los años quedan desde 1980 a 2020
A=readmatrix('SUR.dat');


%% 1.c) se cambian los -9.999 x nan
[i j]= find(A==-999.9); % entrega posiciones del NaN
A(A==-999.9)=NaN;% reemplaza -999.9 por NaN
%entregar tabla con posiciones con NaN
years_nan=A(i,1); % se almacenan los años con nan
mes_nan=j-1; % se almacena el mes, se resta 1 porque los meses parten
% desde la columna 2
tabla01=table(years_nan,mes_nan);


%% 1.d) se crea una nueva matriz B de 2 columnas (fechas,columnas)
% se utilizara una funcion propia llamada "a2c" %01_REA_DAT.m
% La primera columna contendrá los datos fechados a la mitad del mes
B=a2c(A);

%% 1.e) se realiza un plot (figure 1) de la serie B, amplitud versus tiempo
figure(1)
plot(B(:,1),B(:,2),'color','k','linewidth',2)
xlabel('Años','fontsize',15)
ylabel('Extensión de nieve [10^6 km^2]','fontsize',15)
grid on
axis tight
title('Extensión De Nieve en el hemisferio Sur','fontsize',17)


%% 2.a) Se interpolar la 2da columna de B mediante 'spline'

%primero se crea una matriz llamada BsNan sin NaN
BsinNan =  B(all(~isnan(B),2),:); % matriz sin NaN

%datos interpolados
i1=interp1(BsinNan(:,1),BsinNan(:,2),B(:,1),'spline'); 

C=[B(:,1),i1] % La nueve serie interpolada de dos columnas 

tabla03=table(B(:,1),i1) 

figure(2)
subplot(2,1,1)
plot(B(:,1),i1,'color','blue','LineWidth',2)
hold on
plot(B(:,1),B(:,2),'color','k','LineWidth',2) 
xlabel('Tiempo','fontsize',12)
ylabel('Extensión de nieve [10^6 km^2]','fontsize',12)
title('Datos originales ','fontsize',15)
axis tight

subplot (2,1,2)
plot(B(:,1),i1,'color','blue','LineWidth',2)
hold on
plot(B(:,1),B(:,2),'color','k','LineWidth',2) 
xlabel('Tiempo','fontsize',12)
ylabel('Extensión de nieve [10^6 km^2]','fontsize',12)
title('Zoom en Datos originales ','fontsize',15)
axis tight

%se crea una tabla que entrega los datos interpolados
% t1 = datetime(2003,1,15);
% t2 = datetime(2020,12,15);
% t = t1:calmonths(1):t2;
% t=t'
% fecha=t(ubicaciones);
% datinterpol=i1(ubicaciones);
% table(fecha,datinterpol)

%% 3.a) Calcular media, std de la serie SUR interpolada (C)

[mediaC,stdC,~]=estadisticos(C(:,2)); %se usará funcion propia de estadisticos
% media= 24.9420, std=16.6547

%% 3.b)Calcular media y std móvil de la serie (ventana de 61 meses)
% La media móvil estara centrada al principio de la media
[mmC,stdmC]=fnmovil(C(:,2),61) 

% Serie original con su media móvil y su desviación estándar 
% localizando la fecha al principio de la ventana.

l=mmC(:,1); %guardamos en l las fechas de la matriz D
l(432:492)=(NaN);

t=stdmC(:,1);
t(432:492)=(NaN);


%% 3.c) Se grafica media, la desviación estándar, d.estandar móvil y media móvil.
% para graficar la media y std crearemos un vector para estas
 matmed=(zeros(178,1)); 
 matstd=(zeros(178,1));
 %ciclo for para reemplazar valores de la media en todos los datos
 
 for i=1:492
     matmed(i,:)=mediaC;
     matstd(i,:)=stdC;
 end


hold on 
plot(C(:,1),C(:,2),'-','color','k','linewidth',2) 
plot(C(:,1),matmed,'-','color','r','linewidth',3) 
plot(C(:,1),matmed+matstd,':','color','g','linewidth',3)
plot(C(:,1),matmed-matstd,':','color','g','linewidth',3)
plot(C(:,1),l,'-','color','y','linewidth',3)
plot(C(:,1),l+t,':','color','m','linewidth',3)
plot(C(:,1),l-t,':','color','m','linewidth',3)

legend('Datos originales','Media','Media + Dvest','Media - Dvest','medmov','medmov+stdmov','medmov-stdmov','FontSize',12)
title('Extensión De Nieve en el hemisferio Sur','fontsize',17)
set(gcf,'color','w')
xlabel('Tiempo','fontsize',12)
ylabel('Amplitud (km^2)','fontsize',12)
set(gcf,'color','w')
set(gca,'FontSize',12)
grid minor
axis tight
%--------------------------------------------------------------

%% 4.a) Se calcula la mediana, trimediana, cuartiles (1 y 3) y el rango intercuartilico de la serie en C.
q1=quantile(C(:,2),0.25); % primer cuartil
q2=quantile(C(:,2),0.5); % segundo cuartil = mediana
q3=quantile(C(:,2),0.75); % tercer cuartil

mediana=median(C(:,2));

trimean=(q1+2*q2+q3)/4;
riqr=iqr(C(:,2)); % saca rango intercuartil (este tiene %50 de los datos)

%% 4.b) Boxplot de la serie en C

figure()
% boxplot(datos,'labels',{'1980-2021'}) con labels le agregamos titulo bajo
% del boxplot
% Y ademas xlabel('Periodo de tiempo')
% me faltó ylabel('Extension de hielo [10^6 km^2]')
boxplot(C(:,2))
hold on
plot(q1,'*r')
plot(q2,'*c')
plot(q3,'*b')
title('Boxplot extensión de nieve en el hemisferio SUR','FontSize',10)
set(gcf,'color','w')  % color de fondo grafico
set(gca,'FontSize',10)  % tamaño de numeros 
grid on
grid minor      
 % ajusta la figura



%% 4.c) Boxplot periodo de 10 años de C: 1980 89,1990 99, 2000 2009, 2010 2019

b1=C(1:120,2); % de 1 a 120 son 10 años
b2=C(121:240,2);
b3=C(241:360,2);
b4=C(361:480,2);

% Otra forma de hacer lo anterior
c1=C(C>=1980 & C<1990,2); % se indica la condición, y que entregue los datos
c2=C(C>=1990 & C<2000,2);
c3=C(C>=2000 & C<2010,2);
c4=C(C>=2010 & C<2020,2);

M=[b1,b2,b3,b4]; % matriz de años para boxplot

figure(5)
boxplot(M,'labels',{'1980-1989','1990-1999','2000-2009','2010-2019'})
title('Boxplot extensión de nieve en el hemisferio SUR','FontSize',10)
set(gcf,'color','w')  % color de fondo grafico
set(gca,'FontSize',10)  % tamaño de numeros 
grid on
grid minor      
% varios boxplot por separados en un subplot

%% 5.a) La base de datos de SUR.dat contiene un -999.9 mal ubicado en el año 1992
% Interpolación espacial de este dato utilizando interpolación ponderada por pesos diferencial.

% Trabajamos con nuestra matriz original A

%% metodo 2d interpolacion
[f c]=find(isnan(A));
% pos=find(==-999.9);

pondfila=0.02;
pondcol=0.98;

A(15,7)=pondfila*nanmean(A(15,:))+pondcol*nanmean(A(:,7));
A(15,7)=NaN;

%% metodo 2d interpolacion caja

f=15;
c=7;

minicaja=A(f-1:f+1,c-1:c+1); % para caja grande reemplazo todos los valores
                               % de caja chica por NaNs
grancaja=A;
grancaja(f-1:f+1,c-1:c+1)=NaN ;    

pondc=0.99
pondg=0.01

A(f,c)=pondc*nanmean(minicaja(:))+pondg*nanmean(grancaja(:)); % se le coloca : para que de solo 1 valor


