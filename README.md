# Przetwarzanie i przechowywanie opisu siatki trójkątnej na płaszczyźnie
*Mateusz Zając, Błażej Kapkowski*

## Opis programu
Program daje możliwość wczytywania i zapisywania opisów siatek trójkątnych.
Umożliwia wyświetlanie tychże siatek.
Pozwala na wykonanie trzech operacji na siatkach, korzystając z jednej z dwóch
struktur danych - list wierzchołków i trójkątów oraz z Half Edge Data Structure.
Pierwsza operacja to przeglądanie sąsiednich wierzchołków względem wybranego wierzchołka,
kolejna to przeglądanie sąsiednich trójkątów względem trójkąta, ostatnia to przeszukiwanie
kolejnych sąsiednich trójkątów w poszukiwaniu wierzchołka.
Aplikacja ma formę notatnika Jupytera, zawierającego pewne funkcje,
które użytkownik może wywoływać we własnych komórkach. Do wizualizacji wykorzystano narzędzie
napisane przez KN BIT. 

## Wymagania techniczne
- Python 3.10.12
- biblioteki:
  - numpy
  - matplotlib

Wersje bibliotek podane są w pliku `requirements.txt`

## Jak używać
By skorzystać z programu, należy otworzyć notatnik `projekt.ipynb`,
a potem na jego końcu stworzyć komórkę i w niej wywołać odpowiednią funkcję.

W programie często wykorzystywanymi nazwami argumentów są `vertices` i `triangles` (zamiennie z `faces`).
`vertices` to lista wierzchołków, czyli krotek dwóch liczb rzeczywistych (współrzędnych), a `triangles` i `faces` to
lista trójkątów, czyli krotek trzech liczb całkowitych (indeksów wierzchołków trójkąta, numerowanych od 0).

### Wczytywanie z pliku, zapisywanie do niego
Triangulacje wczytuje się za pomocą funkcji `load_from_file(filename)`
gdzie `filename` jest nazwą pliku znajdującego się w katalogu `triangulations`.
Może to być np. `sample_triangulation4.txt`.
Funkcja zwraca krotkę - listę wierzchołków i listę trójkątów.

Triangulację można zapisać do pliku za pomocą funkcji `save_to_file(filename, vertices, triangles)`,
gdzie `filename` to nazwa pliku który pojawi się w katalogu `triangulations`.

Każdy wiersz pliku z triangulacją przyjmuje jedną z dwóch postaci:
- litera `v` i dwie liczby rzeczywiste, wszystko oddzielone spacją, np. `v 1.0 2.0`,
- litera `t` i trzy liczby całkowite, wszystko oddzielone spacją, np. `t 0 1 4`.

### Wyświetlenie triangulacji:
By wyświetlić siatkę, należy wywołać funkcję `draw_triangulation(vertices, triangles, vis)`, gdzie `vis` to obiekt klasy
`Visualizer()`. Funkcja zwraca obiekt tej samej klasy, co `vis`. Należy na nim użyć metody `.show()`.

```py
vis = Visualizer()
vertices, triangles = load_from_file("sample_triangulation4.txt")
vis = draw_triangulation(vertices, triangles, vis)
vis.show()
save_to_file("triangulation_copy.txt", vertices, triangles)
```

### Operacje na listach wierzchołków i trójkątów

#### Pierwsza operacja
Żeby przeszukać siatkę w poszukiwaniu wierzchołków sąsiadujących z innym wierzchołkiem, trzeba użyć funkcji
`find_neighbors(vertex_index, triangles, second_level)`.
`vertex_index` to indeks wierzchołka, którego sąsiadów ma znaleźć funkcja.
`second_level` to argument typu bool. Ustawienie go na True zleca funkcji przeszukanie dwóch kolejnych warstw sąsiadów.
False - tylko jednej. Funkcja zwraca listę składającą się z 2 słowników (pythonowy set()) z indeksami sąsiadów.
Pierwszy element listy to wierzchołki z pierwszej warstwy, a drugi z drugiej, jeśli te znaleziono (w przeciwnym razie,
ten zbiór jest pusty).

W celu wizualizacji tej operacji, należy wywołać funkcję
`find_neighbors_visualize(vertex_index, vertices, triangles, second_level)`.
Zwraca ona krotkę: listę 2 słowników jak powyżej oraz obiekt klasy `Visualizer()`.
Wywołując na nim metodę `.show()` można wyświetlić wizualizację algorytmu, jako obraz.

```py
vertices, triangles = load_from_file("sample_triangulation4.txt")
neighbors, vis = find_neighbors_visualize(5, vertices, triangles, True)
vis.show()
```


#### Druga operacja
Przeszukiwanie trójkątów sąsiadujących z wybranym trójkątem wykonuje się za pomocą funkcji
`find_triangle_neighbors(selected_index, triangles, second_level)`. `selected_index` to indeks
trójkąta, którego sąsiadów się szuka. `second_level` pełni podobną rolę, co w pierwszej operacji - 
to wartość bool, która decyduje ile warstw ma być znalezionych. Funkcja zwraca listę dwóch zbiorów.
Pierwszy zbiór zawiera indeksy trójkątów z pierwszej warstwy, drugi z drugiej.

Wizualizacja: wywołanie funkcji `find_triangle_neighbors_visualize(selected_index, vertices, triangles, second_level)`.
Zwraca listę zbiorów, jak poprzednio, a także obiekt `Visualizer()`.

```py
vertices, triangles = load_from_file("sample_triangulation4.txt")
neighbors, vis = find_triangle_neighbors_visualize(2, vertices, triangles, True)
vis.show()
```

#### Trzecia operacja
Żeby przeszukać siatkę w poszukiwaniu wybranego wierzchołka, iterując po kolejnych, sąsiednich trójkątach,
należy wywołać funkcję `find_triangle_containing_point(start_triangle, point, triangles)`, gdzie
`start_triangle` to indeks trójkąta od którego się zaczyna iterację, `point` - indeks wierzchołka który się szuka.
Funkcja zwraca trójkąt - czyli krotkę indeksów wierzchołków - który zawiera wskazany wierzchołek.
Jeżeli jest w stanie go znaleźć, bo w przeciwnym razie zwraca None.

Żeby zwizualizować tą operację, trzeba wywołać `find_triangle_containing_point_visualize(start_triangle, point,
vertices, triangles, vis)`, gdzie `vis` to wcześniej zainicjalizowany obiekt `Visualizer()`.
Następnie można wykonać `vis.show()`, by wyświetlić wizualizację.

```py
vis = Visualizer()
vertices, triangles = load_from_file("sample_triangulation4.txt")
triangle_found = find_triangle_containing_point_visualize(8, 2, vertices, triangles, vis)
vis.show()
```

### Konstrukcja i wykorzystanie struktury Half Edge
By utworzyć strukturę Half Edge, należy zainicjalizować klasę `HalfEdgeGraph(vertices, faces)`.

#### Pierwsza operacja
Operacja przeszukiwania sąsiednich wierzchołków wykonywana jest przez metodę
`.incidental_vertices(vertex_index, levels)`, gdzie `vertex_index` to wierzchołek z którego się zaczyna, a
`levels` to liczba warstw. Metoda zwraca listę słowników, gdzie każdy słownik odpowiada kolejnej warstwie
i każdy zawiera indeksy wierzchołków z danej warstwy.

Żeby zwizualizować operację, trzeba wykonać `.incidental_vertices_visualize(vertex_index, levels, visualizer)`,
gdzie `visualizer` to wcześniej zainicjalizowany obiekt `Visualizer()`.

Zamiast metody `.show()`, można zastosować metodę `.show_gif(delay)`, która
przedstawi kolejne kroki wykonywania algorytmu, każdy krok pokazany co `delay` sekund.

```python
vis = Visualizer()
vertices, triangles = load_from_file("sample_triangulation4.txt")
vis = draw_triangulation(vertices, triangles, vis)
half_edges = HalfEdgeGraph(vertices, triangles)
neighbors, vis = half_edges.incidental_vertices_visualize(5, 2, vis)
vis.show_gif(500)
```

#### Druga operacja
Przeglądanie sąsiednich trójkątów - używa się metody `.incidental_triangles(face_index, levels)`.
`face_index` - indeks startowego trójkąta, `levels` - liczba warstw.
Funkcja zwraca listę słowników, każdy odpowiada kolejnej warstwie i zawiera indeksy trójkątów z tej warstwy.

Wizualizacja - metoda `.incidental_triangles_visualize(face_index, levels, visualizer)`.
Sposób wywoływania analogiczny jak w pierwszej operacji.

```python
vis = Visualizer()
vertices, triangles = load_from_file("sample_triangulation4.txt")
vis = draw_triangulation(vertices, triangles, vis)
half_edges = HalfEdgeGraph(vertices, triangles)
neighbors, vis = half_edges.incidental_triangles_visualize(2, 2, vis)
vis.show_gif(500)
```

#### Trzecia operacja
Przeszukiwanie trójkąta zawierającego dany wierzchołek - należy użyć metody
`.seek_vertex(face_index, vertex_index)`. `face_index` - początkowy trójkąt,
`vertex_index` - szukany wierzchołek. Metoda zwraca indeks znalezionego trójkąta,
jeśli taki istnieje. Jeśli nie istnieje, zwraca None.

Wizualizacja - metoda `.seek_vertex_visualize(face_index, vertex_index, visualizer)`.
Wywoływanie - analogiczne.

```python
vis = Visualizer()
vertices, triangles = load_from_file("sample_triangulation4.txt")
vis = draw_triangulation(vertices, triangles, vis)
half_edges = HalfEdgeGraph(vertices, triangles)
found_triangle, vis = half_edges.seek_vertex_visualize(2, 2, vis)
vis.show_gif(500)
```