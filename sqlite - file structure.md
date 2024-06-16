#sqlite
https://link.springer.com/content/pdf/10.1007/978-3-030-98467-0_5.pdf

SQLite is a single-file database engine, i.e., all tables are managed in only one file
on disk
- First 100 bytes is the [header](https://www.sqlite.org/fileformat.html#database_header)
- Beyond this, the db is divided into pages of equal size (page size can be found in the header). Note that the database header is part on the first page and does not show on other pages => The file size is always a multiple of the page size

### First page

- First 100 bytes is the header
- Rest of the page is the database schema. Necessary information such as the root page numbers, column names, and column types of the tables are stored here, in a data structure called **SQLite_Master Table**.

The schema table contains all database objects in the database and the statement used to create each object. With the schema table’s help, all table names, the corresponding column names and data types can be determined.


Data is stored as a [[B+ Tree]] => records are only on leaf pages

### other vision

[Database File Format (sqlite.org)](https://www.sqlite.org/fileformat.html)

- Juxtaposed pages: total size = page_size * page_no
- To go to the beginning of a page, seek (page_no -1) * page_size
- 4 type of pages:
```rust 
#[derive(Debug, PartialEq)]
#[binrw]
pub enum PageType {
    #[brw(magic = 2u8)]
    InteriorIndex,
    #[brw(magic = 5u8)]
    InteriorTable,
    #[brw(magic = 10u8)]
    LeafIndex,
    #[brw(magic = 13u8)]
    LeafTable,
}
```

- B-Tree structure: 
	- B+ tree for Table
	- For index, Interior and Leaf both contain integer keys referencing the record

- On page 1, database header (100 bytes), contains info such as page_size, text_encoding
- Each page starts with a header (first page is smaller because of db header). Header contains page_size, nb_of cells... For interior, also contains right most pointer (points to the last child page)
- After header, cell_pointer_array: offsets to cells
- Interior cells contain references to child pages 
- Leaf cells contain actual records
- Note: cells can overflow, in this case, last part is pointer to next page containing rest of content

First page contains the sqlite_schema [The Schema Table (sqlite.org)](https://sqlite.org/schematab.html)
It contains records for all objects. We can retrieve the root page pointing to a given object, seek the page and then parse it to get the records. 
