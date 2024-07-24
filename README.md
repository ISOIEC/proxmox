# proxmox
Добавление температур в статистику

За основу взята статья https://voronin.one/all/proxmox-temperatura-processora/

Для начала установить lm-sensors для получения данных из командной строки, соединяемся по ssh или через консоль, далее:

	apt update
	apt install lm-sensors
  
Проверка командой:

	sensors
  
  ![image](https://github.com/user-attachments/assets/f4a68fde-55a3-4caa-969b-fe958a4d17d2)

Если всё прошло успешно, то дальше редактируем файл  /usr/share/perl5/PVE/API2/Nodes.pm

	nano  /usr/share/perl5/PVE/API2/Nodes.pm
  
ищем словосочетание «my $dinfo» (поиск через ctrl + w)
Перед добавляем $res->{thermalstate} = `sensors`; (ctrl + o сохранить, ctrl + x закрыть) Должно получиться

![image](https://github.com/user-attachments/assets/659ee4e6-65b0-4535-9ebc-ecea0df9b36e)

Отредактируем область вывода в файле /usr/share/pve-manager/js/pvemanagerlib.js
  nano /usr/share/pve-manager/js/pvemanagerlib.js
и ищем «widget.pveNodeStatus»

редактируем значние прод себя, чем больше ядер тем больше значение в height

    height: 400,
    bodyPadding: '20 15 20 15',
Далее ищем Manager Version

После этой секции дописываем следующее, в отличии от оригинала ядра и подпись добавляются сами не важно сколько их 1 или 20

	{
            itemId: 'thermal',
            colspan: 2,
            printBar: false,
            title: gettext('CPU Thermal State'),
            textField: 'thermalstate',
            renderer: function(value) {
                const data = value.split('\n').filter(val => val.includes('Core ')).reduce((acc, current, index) => {
                const temperature = current.match(/([\d\.]+)Â/)[1];

                acc += `Core ${index + 1}: ${temperature} ℃  | `;
                 return acc;
                        }, '').slice(0, -2);
  
			return `<div style="width: 70%; text-align: right;">${data}</div>`
	},
 
Выглядеть должно следудующим образом:
![image](https://github.com/user-attachments/assets/b15947e2-30c8-42be-8291-5c379239aed6)

Перезапцускаем службу командой:
	systemctl restart pveproxy
Проверяем в web что получилось:
![image](https://github.com/user-attachments/assets/f82df303-eec0-4711-97bb-fde9ca8a9b60)

Это не финальный вариант отображения, после обовлю как наведу красоту
