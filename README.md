# proxmox добавление температур в статистику

За основу взята [статья](https://voronin.one/all/proxmox-temperatura-processora/).

Спасибо за помощь с js [shamilfrontend](https://github.com/shamilfrontend).

 **В отличии от оригинала ядра добавляются сами не важно сколько их 1 или 20**

Для начала установить lm-sensors для получения данных из командной строки, соединяемся по ssh или через консоль, далее:

	apt update
	apt install lm-sensors
  
Проверка командой:

	sensors
  
  ![image](https://github.com/user-attachments/assets/f4a68fde-55a3-4caa-969b-fe958a4d17d2)

Если всё прошло успешно, то дальше редактируем файл  /usr/share/perl5/PVE/API2/Nodes.pm

	nano  /usr/share/perl5/PVE/API2/Nodes.pm
  
ищем словосочетание «my $dinfo» (поиск через ctrl + w)
Перед добавляем $res->{thermalstate} = `sensors`; (ctrl + o сохранить, ctrl + x закрыть) Должно получиться:

![image](https://github.com/user-attachments/assets/659ee4e6-65b0-4535-9ebc-ecea0df9b36e)

Отредактируем область вывода в файле /usr/share/pve-manager/js/pvemanagerlib.js

  	nano /usr/share/pve-manager/js/pvemanagerlib.js
  
и ищем «widget.pveNodeStatus»

редактируем значние прод себя, чем больше ядер тем больше значение в height:

    height: 400,
    bodyPadding: '20 15 20 15',
Далее ищем Manager Version

После этой секции дописываем следующее:

        {
            itemId: 'thermal',
            colspan: 2,
            printBar: false,
            title: gettext('CPU Thermal State'),
            textField: 'thermalstate',
        renderer: function(value) {
                const data = value.split('\n').filter(val => val.includes('Core ')).reduce((acc, current, index) => {
                const temperature = current.match(/([\d\.]+)Â/)[1];
                        acc += `<div style="border: 1px solid #111; padding: 2px;">Core ${index + 1}: ${temperature} ℃</div>`;
                return acc;
                        }, '').slice(0, -2);
                return `<div style="display: grid; grid-template-columns: repeat(7, 1fr); grid-gap: 8px;">${data}</div>`
        },
        },
    ],

 
Выглядеть должно следудующим образом:

![image](https://github.com/user-attachments/assets/bc4d59f1-a3a7-443f-8cab-b108f5f69440)



Перезапцускаем службу командой:
	systemctl restart pveproxy
Проверяем в web что получилось:

![image](https://github.com/user-attachments/assets/59a15a4c-c317-47a7-9ef5-f481d16705c4)

