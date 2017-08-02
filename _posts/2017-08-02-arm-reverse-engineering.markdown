---
layout: post
title:  "Inżynieria wsteczna Portalu Azure"
date:   2017-08-02 02:14:52 +0200
categories: azure arm
---
Obecnie jest dosyć dużo źródeł informacji nt. wdrożeń szablonów ARM. Chociażby [ARM template reference][arm-template-reference] lub [Azure Quickstart Templates][azure-quickstart-templates], żeby wspomnieć tylko te najbardziej oczywiste. Ale co w sytuacji kiedy usługa jest dopiero w fazie preview lub z jakichś powodów dostępna dokumentacja nie jest wystarczająca, żeby stworzyć przy jej pomocy poprawny szablon ARM? Cóż, wtedy zostaje nam inżynieria wsteczna portalu. Poniżej opisuję cały proces wyszukiwania tych informacji na przykładzie Bot Service.

# Przygotowanie do przechwycenia JSONa
W momencie tworzenia tego wpisu nie ma żadnych informacji na temat Bot Service w żadnym z wyżej wymienionych źródeł. Jeśli więc chcemy wdrożyć Bot Service za pomocą szablonu pozostaje nam "wydrzeć" te dane Portalu Azure. Aby to zrobić, należy przejść procedurę tworzenia obiektu i zatrzymać się przed ostatnim krokiem czyli kliknięciem przycisku Create. W tym momencie włączamy Developer Toolsy w używanej przeglądarce, w moim przypadku jest to Opera. Przechodzimy na zakładkę Network, upewniamy się, że jest pusta, żeby łatwiej było nam szukać interesujących nas requestów. Następnie klikamy przycisk Create i czekamy aż obiekt zostanie stworzony.

# Wyszukiwanie requestów
Kiedy portal poinformuje nas, że tworzenie obiektu zakończyło się sukcesem, można wyłączyć śledzenie requestów. W zdecydowanej większości przypadku portal wysyła informacje w formie obiektów JSON do znajdujących się pod spodem API. Trzeba je tylko znaleźć, aby ułatwić sobie proces szukania w oknie Filter można wpisać method:POST lub method:PUT. Usuniemy w ten sposób z listy pozycje w których na pewno nie znajdziemy nic ciekawego.

Przeszukując listę trafiamy na request do adresu https://management.azure.com/api/invoke?_=0 gdzie w sekcji Request Payload widzimy poniższe dane.

![request]({{ site.url }}/assets/02_devtools.png) ![request-raw]({{ site.url }}/assets/03_devtools.png)

# Konwersja na szablon ARM
Po przekopiowaniu ich do VS Code'a i sformatowaniu stają się czytelniejsze. W przypadku Bot Service sytuacja jest o tyle prosta, że całą strukturę dostajemy praktycznie na tacy. Widać wyraźnie, że JSON przypomina standardowy szablon ARM.

![json]({{ site.url }}/assets/04_vscode.png)

Wystarczy więc wrzucić kluczową sekcję do szablonu, przygotować odpowiednio parametry i ewentualnie wprowadzić pożądane zmiany, jak np. ustawienie dziedziczenia lokalizacji po Resource Groupie.

![arm]({{ site.url }}/assets/05_vscode.png)

# Wdrożenie szablonu ARM
Po przygotowaniu szablonu ARM pozostaje już tylko stworzyć skrypt wywołujący jego wdrożenie jak np. ten poniżej i można sprawdzać czy wdrożenie się powiedzie.

![deploy.ps1]({{ site.url }}/assets/06_vscode.png)

Nie zawsze jest tak prosto jak w przypadku Bot Service. Czasami trzeba skleić nasz szablon z kilku requestów, innym razem może się okazać, że portal nie wysyła w ogóle żadnych informacji nt. tworzonego obiektu w formie jawnego JSONa. W takiej sytuacji najprawdopodobniej zostaje nam niestety czekać aż się to zmieni lub pojawią się jakieś informacje w oficjalnej dokumentacji lub przykłady na Azure Quickstart Templates.

[arm-template-reference]: https://docs.microsoft.com/en-us/azure/templates/
[azure-quickstart-templates]: https://github.com/Azure/azure-quickstart-templates
