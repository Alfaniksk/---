import json
import os
from datetime import datetime

class Book:
    """Класс для представления книги"""
    
    def __init__(self, title, author, genre, year, description, book_id=None):
        self.id = book_id or str(datetime.now().timestamp())
        self.title = title
        self.author = author
        self.genre = genre
        self.year = year
        self.description = description
        self.is_read = False
        self.is_favorite = False
    
    def to_dict(self):
        """Преобразует объект книги в словарь для сохранения в JSON"""
        return {
            'id': self.id,
            'title': self.title,
            'author': self.author,
            'genre': self.genre,
            'year': self.year,
            'description': self.description,
            'is_read': self.is_read,
            'is_favorite': self.is_favorite
        }
    
    @classmethod
    def from_dict(cls, data):
        """Создает объект книги из словаря"""
        book = cls(
            title=data['title'],
            author=data['author'],
            genre=data['genre'],
            year=data['year'],
            description=data['description'],
            book_id=data['id']
        )
        book.is_read = data['is_read']
        book.is_favorite = data['is_favorite']
        return book
    
    def __str__(self):
        status = "✓ Прочитана" if self.is_read else "✗ Не прочитана"
        favorite = "★ В избранном" if self.is_favorite else ""
        return f"ID: {self.id[:8]}... | {self.title} | {self.author} | {self.year} | {self.genre} | {status} {favorite}"


class Library:
    """Класс для управления библиотекой"""
    
    def __init__(self, filename='library.json'):
        self.filename = filename
        self.books = []
        self.load_from_file()
    
    def add_book(self, title, author, genre, year, description):
        """Добавляет новую книгу в библиотеку"""
        book = Book(title, author, genre, year, description)
        self.books.append(book)
        self.save_to_file()
        print(f"\n✅ Книга '{title}' успешно добавлена в библиотеку!")
        return book
    
    def get_all_books(self):
        """Возвращает список всех книг"""
        return self.books
    
    def get_book_by_id(self, book_id):
        """Находит книгу по ID"""
        for book in self.books:
            if book.id == book_id:
                return book
        return None
    
    def remove_book(self, book_id):
        """Удаляет книгу из библиотеки по ID"""
        book = self.get_book_by_id(book_id)
        if book:
            self.books.remove(book)
            self.save_to_file()
            print(f"\n✅ Книга '{book.title}' удалена из библиотеки")
            return True
        print("\n❌ Книга с таким ID не найдена")
        return False
    
    def toggle_read_status(self, book_id):
        """Изменяет статус прочтения книги"""
        book = self.get_book_by_id(book_id)
        if book:
            book.is_read = not book.is_read
            self.save_to_file()
            status = "прочитана" if book.is_read else "не прочитана"
            print(f"\n✅ Статус книги '{book.title}' изменен на '{status}'")
            return True
        print("\n❌ Книга с таким ID не найдена")
        return False
    
    def toggle_favorite(self, book_id):
        """Добавляет или удаляет книгу из избранного"""
        book = self.get_book_by_id(book_id)
        if book:
            book.is_favorite = not book.is_favorite
            self.save_to_file()
            status = "добавлена в избранное" if book.is_favorite else "удалена из избранного"
            print(f"\n✅ Книга '{book.title}' {status}")
            return True
        print("\n❌ Книга с таким ID не найдена")
        return False
    
    def get_favorite_books(self):
        """Возвращает список избранных книг"""
        return [book for book in self.books if book.is_favorite]
    
    def get_unread_books(self):
        """Возвращает список непрочитанных книг (для рекомендаций)"""
        return [book for book in self.books if not book.is_read]
    
    def search_books(self, keyword):
        """Ищет книги по ключевому слову в названии, авторе или описании"""
        keyword = keyword.lower()
        results = []
        for book in self.books:
            if (keyword in book.title.lower() or 
                keyword in book.author.lower() or 
                keyword in book.description.lower()):
                results.append(book)
        return results
    
    def filter_by_genre(self, genre):
        """Фильтрует книги по жанру"""
        return [book for book in self.books if book.genre.lower() == genre.lower()]
    
    def filter_by_status(self, is_read):
        """Фильтрует книги по статусу прочтения"""
        return [book for book in self.books if book.is_read == is_read]
    
    def sort_books(self, key='title', reverse=False):
        """Сортирует книги по указанному ключу"""
        valid_keys = {
            'title': lambda x: x.title.lower(),
            'author': lambda x: x.author.lower(),
            'year': lambda x: x.year
        }
        
        if key not in valid_keys:
            print(f"❌ Неверный ключ сортировки. Используйте: {', '.join(valid_keys.keys())}")
            return self.books
        
        return sorted(self.books, key=valid_keys[key], reverse=reverse)
    
    def save_to_file(self):
        """Сохраняет данные библиотеки в файл"""
        data = [book.to_dict() for book in self.books]
        try:
            with open(self.filename, 'w', encoding='utf-8') as f:
                json.dump(data, f, ensure_ascii=False, indent=2)
        except Exception as e:
            print(f"❌ Ошибка при сохранении файла: {e}")
    
    def load_from_file(self):
        """Загружает данные библиотеки из файла"""
        if not os.path.exists(self.filename):
            return
        
        try:
            with open(self.filename, 'r', encoding='utf-8') as f:
                data = json.load(f)
                self.books = [Book.from_dict(book_data) for book_data in data]
            print(f"✅ Загружено {len(self.books)} книг из файла")
        except Exception as e:
            print(f"❌ Ошибка при загрузке файла: {e}")
            self.books = []


class ConsoleApp:
    """Консольный интерфейс приложения"""
    
    def __init__(self):
        self.library = Library()
    
    def print_header(self, text):
        """Выводит заголовок"""
        print("\n" + "="*60)
        print(f"   {text}")
        print("="*60)
    
    def print_menu(self):
        """Выводит главное меню"""
        self.print_header("T-БИБЛИОТЕКА - Управление личной библиотекой")
        print("1. 📚 Добавить книгу")
        print("2. 📖 Просмотреть все книги")
        print("3. 🔍 Поиск книг")
        print("4. ⭐ Избранные книги")
        print("5. 📋 Непрочитанные книги (рекомендации)")
        print("6. ✏️ Изменить статус книги")
        print("7. ❤️ Добавить/удалить из избранного")
        print("8. 🗑️ Удалить книгу")
        print("9. 💾 Сохранить данные")
        print("0. 🚪 Выйти")
    
    def add_book_menu(self):
        """Меню добавления книги"""
        self.print_header("ДОБАВЛЕНИЕ НОВОЙ КНИГИ")
        
        title = input("Название книги: ").strip()
        if not title:
            print("❌ Название не может быть пустым")
            return
        
        author = input("Автор: ").strip()
        if not author:
            print("❌ Автор не может быть пустым")
            return
        
        genre = input("Жанр: ").strip()
        year = input("Год издания: ").strip()
        
        try:
            year = int(year) if year else None
        except ValueError:
            print("❌ Год должен быть числом")
            return
        
        description = input("Краткое описание: ").strip()
        
        self.library.add_book(title, author, genre, year, description)
    
    def display_books(self, books, title="КНИГИ"):
        """Отображает список книг"""
        self.print_header(title)
        
        if not books:
            print("📭 Книги не найдены")
            return
        
        for i, book in enumerate(books, 1):
            print(f"\n{i}. {book}")
            print(f"   📝 {book.description[:100]}{'...' if len(book.description) > 100 else ''}")
    
    def view_books_menu(self):
        """Меню просмотра книг с сортировкой и фильтрацией"""
        self.print_header("ПРОСМОТР КНИГ")
        
        print("Фильтры:")
        print("1. Все книги")
        print("2. По жанру")
        print("3. Только прочитанные")
        print("4. Только непрочитанные")
        print("0. Назад")
        
        choice = input("\nВыберите фильтр: ").strip()
        
        books = []
        if choice == '1':
            books = self.library.get_all_books()
        elif choice == '2':
            genre = input("Введите жанр: ").strip()
            books = self.library.filter_by_genre(genre)
        elif choice == '3':
            books = self.library.filter_by_status(True)
        elif choice == '4':
            books = self.library.filter_by_status(False)
        else:
            return
        
        if not books:
            print("📭 Книги не найдены")
            return
        
        print("\nСортировка:")
        print("1. По названию")
        print("2. По автору")
        print("3. По году издания")
        print("0. Без сортировки")
        
        sort_choice = input("\nВыберите сортировку: ").strip()
        
        sort_map = {
            '1': 'title',
            '2': 'author',
            '3': 'year'
        }
        
        if sort_choice in sort_map:
            books = self.library.sort_books(sort_map[sort_choice])
        
        self.display_books(books, f"РЕЗУЛЬТАТЫ ({len(books)} книг)")
    
    def search_menu(self):
        """Меню поиска книг"""
        self.print_header("ПОИСК КНИГ")
        
        keyword = input("Введите ключевое слово для поиска: ").strip()
        if not keyword:
            print("❌ Введите ключевое слово")
            return
        
        results = self.library.search_books(keyword)
        self.display_books(results, f"РЕЗУЛЬТАТЫ ПОИСКА: '{keyword}' ({len(results)} книг)")
    
    def run(self):
        """Главный цикл приложения"""
        while True:
            self.print_menu()
            choice = input("\nВыберите действие: ").strip()
            
            if choice == '1':
                self.add_book_menu()
            elif choice == '2':
                self.view_books_menu()
            elif choice == '3':
                self.search_menu()
            elif choice == '4':
                self.display_books(self.library.get_favorite_books(), "ИЗБРАННЫЕ КНИГИ")
            elif choice == '5':
                self.display_books(self.library.get_unread_books(), "РЕКОМЕНДАЦИИ (НЕПРОЧИТАННЫЕ КНИГИ)")
            elif choice == '6':
                self.display_books(self.library.get_all_books(), "ВЫБЕРИТЕ КНИГУ")
                book_id = input("\nВведите ID книги: ").strip()
                self.library.toggle_read_status(book_id)
            elif choice == '7':
                self.display_books(self.library.get_all_books(), "ВЫБЕРИТЕ КНИГУ")
                book_id = input("\nВведите ID книги: ").strip()
                self.library.toggle_favorite(book_id)
            elif choice == '8':
                self.display_books(self.library.get_all_books(), "ВЫБЕРИТЕ КНИГУ ДЛЯ УДАЛЕНИЯ")
                book_id = input("\nВведите ID книги: ").strip()
                confirm = input("Вы уверены? (да/нет): ").strip().lower()
                if confirm == 'да':
                    self.library.remove_book(book_id)
            elif choice == '9':
                self.library.save_to_file()
                print("✅ Данные сохранены")
            elif choice == '0':
                print("\n👋 До свидания! Данные сохранены.")
                self.library.save_to_file()
                break
            else:
                print("❌ Неверный выбор. Попробуйте снова.")
            
            input("\nНажмите Enter для продолжения...")


if __name__ == "__main__":
    app = ConsoleApp()
    app.run()# ---
