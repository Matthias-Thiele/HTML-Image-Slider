# HTML-Image-Slider

Da ich auf meiner Hauptseite viele Bilder in Galerieform habe, bin ich mit dem Standard-Slider unzufrieden. Er ist nicht einfach zu bedienen und zeigt die Bilder nicht leicht durchblätterbar und in voller Bildschirmgröße an. Ich habe mir deshalb mal einige Slider aus dem Internet angesehen und habe nicht das richtige gefunden. Zum einen arbeiten einige Slider nicht sauber mit dem Firefox Browser zusammen. Insbesondere die reinen CSS Slider machen so viele Verrenkungen, dass es mich nicht wundert, das es an allen Ecken und Kanten hakt. Ich denke, dass einige Web Entwickler nur noch auf Chrome testen. Zum anderen muss man bei fast allen Slidern die html Seite anpassen. Ich habe keinen generischen Slider gesehen, der über Parameter gesteuert werden kann.

Im Nachhinein muss ich erkennen, dass ich länger nach einem brauchbaren Slider gesucht habe als die Eigenentwicklung Zeit gekostet hat. Wichtig für mich war, dass der Slider die Bilder aus dem WordPress Media Pool anzeigen kann, er sollte nicht für jede Galerie eine eigene Web Seite benötigen und er sollte per Maus und Tastatur gesteuert werden können. Und das Wichtigste: er sollte die Bilder in voller Größe anzeigen. Ein reiner CSS Slider war für mich hingegen nicht erstrebenswert. Wegen der Parametrisierbarkeit benötige ich ohnehin JavaScript. Ein einfacher CSS Aufbau hingegen ist ein echter Vorteil.

Das komplette Projekt besteht nur aus einer Datei, sie kann von meinem Github Repository herunter geladen werden: https://github.com/Matthias-Thiele/HTML-Image-Slider

Der Grundgedanke beruht wie bei vielen Slidern darauf, dass die Bilder übereinander gestapelt werden und immer nur ein Bild angezeigt wird. Das kann man über CSS leicht mit „display: none;“ steuern. Der Image Bereich ist bei mir zum Start leer, er wird beim Aufruf der Seite aus der Parameterliste heraus gefüllt.

          <div id="imagelist">
          </div>

Der JavaScript Code dazu ist überschaubar. Die Bilder können entweder durchnummeriert kommen oder mit einem beschreibenden Namen. Es wird ein IMG Element erzeugt, der Pfad zum Bild eingefügt und das Element in die imagelist eingehangen.

for (var i = imgStart; i <= imgEnd; i++) {
        var img = document.createElement("img");
        var imgSrc;

        if (descriptionName) {
            imgSrc = this.startPath + imgName + this.description[i] + "." + imgExt;
        } else {
            var num = (i < 10) ? (pendingZero + i) : i;
            imgSrc = this.startPath + imgName + num + "." + imgExt;
        }

        img.src = imgSrc;
        if (i === imgStart) {
            img.className = "active";
        }
        listRoot.appendChild(img);
        this.items.push(img);
    }

Die Statuszeile überlagert halb-transparent den unteren Bildteil. Vielleicht erweitere ich es später mal so, dass man sie auch ausblenden kann. In den meisten Fällen wird sie aber nicht stören.

          <nav class="slider-nav">
              <button class="previous" data-key="true" style="width:52%; text-align: right">
              <span>
                  <i>&lt;</i>
              </span>
            </button>
            <button class="next" data-key="false" style="text-align: left">
              <span>
                <i>&gt;</i>
              </span>
            </button>
              <span id="counter" style="float:right; width: 100pt; padding: 2pt;">1/6</span>
          </nav>
        </div>

Die Bild-Weiterschaltung läuft im Kreis. Wenn man am letzten Bild angekommen wird, wird als nächstes das erste Bild angezeigt und umgekehrt. Damit man die Orientierung nicht verliert und X-mal im Kreis läuft, wird in der Statuszeile angezeigt, wie viele Bilder es gibt und auf welchem Bild man gerade steht.

            Slider.prototype.advance = function(leftRight) {
                this.items[this.count].classList.remove('active');
                this.count += (leftRight) ? -1 : 1;
                
                if (this.count < 0) {
                    this.count = this.items.length -1;
                } else if (this.count >= this.items.length) {
                    this.count = 0;
                }
                
                this.items[this.count].classList.add('active');
                this.updateView();
            };
            
            Slider.prototype.updateView = function() {    
                var status = document.getElementById("counter");
                status.innerText = " " + (this.count + 1) + " / " + this.items.length; 
                
                var desc = document.getElementById("description");
                if (this.count >= this.description.lengt || !this.description[this.count]) {
                    desc.style = "display: none";
                } else {
                    desc.innerText = this.description[this.count];
                    desc.style = "display: inline-block";
                }
            };

Die Tastatursteuerung ist einfach. Es gibt nur zwei Kommandos – vorwärts und zurück. Diese können über Pfeil links/ rechts und über Bild hoch/ runter ausgelöst werden.

            Slider.prototype.keyPress = function(event) {
                event = event || window.event;
                var slider = document.querySelector('.next').slider;
                var keyCode = event.keyCode;
                
                if (keyCode === 33 || keyCode === 37) {
                  slider.advance(true);
                } else if (keyCode === 34 || keyCode === 39) {
                  slider.advance(false);
                }
            };

Die HTML Datei enthält keine Informationen zu den Bildern. Diese wird beim Aufruf der Seite über Parameter mitgegeben. Somit kann man die Seite irgendwo hinterlegen und aus beliebigen Anwendungen heraus aufrufen. Innerhalb der Web Seite kann man optional den Start des Pfades der Quelle hinterlegen. Damit kann man erzwingen, dass ein bestimmter Bereich nicht einfach verlassen werden kann und zusätzlich ist dann der Parametersatz etwas kürzer, da dieser Teil nicht jedes mal mit angegeben werden muss.

            var slider = new Slider("imagelist", "https://mmth.de/wp-content/uploads/");

Der erste Parameter gibt die id des DIV Elements an welches die Imageliste enthalten soll und der zweite Parameter den Start der URL jedes Bildes. Man kann diesen auch auf einen Leerstring setzen (nicht einfach weglassen).

http://localhost:8080/SliderTest/slider.html?name=2023/09/&ext=jpg&desc=Mainau1~Mainau2~Mainau3~Mainau5

Folgende Parameter gibt es:
name	Enthält den festen Namensanteil des Bildes. Bei durchnummerierten Bildern wird die Nummer am Ende automatisch angefügt. Falls der Startpfad aus der html Datei noch um Unterverzeichnisse ergänzt werden muss, kommen diese hier vor den Namen. Bei Bildern deren Name aus der Beschreibung generiert wird, kann man hier den Verzeichnispfad hinterlegen.	name=Mainau
name=2023/Mainau
start	Bei durchnummerierten Bilder wird hier die Nummer des ersten Bilds angegeben. Im Normalfall wird das 1 sein, das muss aber nicht zwingend so sein. Wenn man nur eine Teilsequenz anzeigen möchte, kann man hier auch eine höhere Zahl angeben.
Falls die Bilder nicht Bild1, Bild2, .., Bild9, Bild10 durchnummeriert sind, sondern mit fester Nummernbreite Bild01, Bild 02, .., Bild09, Bild10 bezeichnet wurden, muss man beim Startwert auch eine führende 0 einfügen: start=01 	start=1
end	Hier wird die Nummer des letzten Bildes eingetragen	end=5
ext	Alle Bilddateien müssen vom gleichen Typ sein, da man nur eine Extension (ohne Punkt) angeben kann. Der Name setzt sich dann aus Startpfad + Name + Nummer + Punkt + Extension zusammen.	ext=jpg
desc	Bei durchnummerierten Bildern ist der Beschreibungsteil optional. Wenn er vorhanden ist, wird die Beschreibung zu jedem Bild oben links eingeblendet. Bei Bilddateinamen aus der Beschreibung muss dieser Parameter eine Liste aller Namen der Bilddateien enthalten.	desc=Haus~Auto~Boot
