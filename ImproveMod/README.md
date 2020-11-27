 * UWAGA!!!
 * Należy zmodyfikować odpowiednie pliki, które zostały w tym pliku opisane!
 *
 * Modyfikacja nie jest w ostatecznej wersji :) Można ją zmieniać / poprawiać, lecz zostawić autora skryptu.
 * Z góry dzięki za skorzystanie autorskich modyfikacji!!
 *
 * @Author KarajuSs
 
// Ścieżka: src/games/stendhal/server/entity/item/Item.java
Szukamy:
```
		entity.addAttribute("autobind", Type.FLAG, (byte) (Definition.HIDDEN | Definition.VOLATILE));
```

Dodajemy pod:
```
		entity.addAttribute("max_improves", Type.INT, Definition.VOLATILE);
		entity.addAttribute("improve", Type.INT);
```

Szukamy:
```
	public static int getDefaultAttackRate() {
		return DEFAULT_ATTACK_RATE;
	}
```

Dodajemy pod:
```
	public int getImproves() {
		if(has("improve")) {
			return getInt("improve");
		}
		return 0;
	}
	public int getMaxImproves() {
		if(has("max_improves")) {
			return getInt("max_improves");
		}
		return 0;
	}
	public boolean isMaxImproved() {
		if (getImproves() == getMaxImproves()) {
			return true;
		}
		return false;
	}
```

// Ścieżka: src/games/stendhal/server/core/config/ItemsXMLLoader.java
// Jeśli istenieje zdeklarowana wartość 'condition'
Szukamy:
```
		if (qName.equals("status_resist")) {
			this.resistances.put(attrs.getValue("type"), Double.valueOf(attrs.getValue("value")));
			this.activeSlots = attrs.getValue("slots");
		} else {
			attributes.put(qName, attrs.getValue("value"));
		}
```

Zamieniamy na:
```
		if (qName.equals("status_resist")) {
			this.resistances.put(attrs.getValue("type"), Double.valueOf(attrs.getValue("value")));
			this.activeSlots = attrs.getValue("slots");
		} else if (qName.equals("max_improves")) {
			attributes.put(qName, attrs.getValue("value"));
			attributes.put("improve", "0");
		} else {
			attributes.put(qName, attrs.getValue("value"));
		}
```

// Jeśli zdeklarowana wartość 'condition' nie istnieje
Szukamy:
```
		} else if (qName.equals("attributes")) {
```
		
Zamieniamy całą funkcję JEŻELI na:
```
		} else if (qName.equals("attributes")) {
			attributesTag = true;
		} else if (attributesTag) {
			boolean conditionMet = true;

			// allow attributes to be disabled with system properties
			String condition = attrs.getValue("condition");
			if (condition != null) {
				if (condition.startsWith("!")) {
					condition = new StringBuilder(condition).deleteCharAt(0).toString();
					conditionMet = System.getProperty(condition) == null;
				} else {
					conditionMet = System.getProperty(condition) != null;
				}
			}

			if (conditionMet) {
				if (qName.equals("status_resist")) {
					this.resistances.put(attrs.getValue("type"), Double.valueOf(attrs.getValue("value")));
					this.activeSlots = attrs.getValue("slots");
				} else if (qName.equals("max_improves")) {
					attributes.put(qName, attrs.getValue("value"));
					attributes.put("improve", "0");
				} else {
					attributes.put(qName, attrs.getValue("value"));
				}
			}
		}
```

// Ścieżka: src/games/stendhal/server/core/engine/transformer/ItemTransformer.java
Szukamy tablicę:
```
		final String[] individualAttributes
```

Dodajemy w tablicy przed "logid": `"improve"`


// Ścieżka: data/conf/items.xsd
Szukamy:
```
		<element name="menu" type="Q1:attribute" minOccurs="0"/>
```
		
Dodajemy pod:
```
		<element name="max_improves" type="Q1:attribute" minOccurs="0" maxOccurs="1"/>
```


* Cieszymy się ze skryptu umożliwiającego ulepszanie przedmiotów :) *

// Mini poradnik jak dodać przedmiot jako możliwy do ulepszenia:
1. Otwieramy przykładowo plik konfiguracyjny `'armors.xml'`.
2. Szukamy przedmiotu `'skórzana zbroja'`.
3. Szukamy deklarację implementacji objektu: `<implementation class-name="games.stendhal.server.entity.item.Item"/>`
	i zamieniamy na nowy objekt: `<implementation class-name="games.stendhal.server.entity.item.ImprovableItem"/>`
4. Szukamy wartość obrony: `<def value="3"/>`
	i dodajemy pod nową wartość maksymalnej liczby ulepszeń tego przedmiotu: `<max_improves value="2"/>`
