# Relationship-Design

Q 1- 
server.js
const express = require("express");
const mongoose = require("mongoose");
const dotenv = require("dotenv");

dotenv.config();
const app = express();
app.use(express.json());

// MongoDB Connection
mongoose.connect(process.env.MONGO_URI, {
  useNewUrlParser: true,
  useUnifiedTopology: true,
}).then(() => console.log("Connected to MongoDB"))
  .catch(err => console.error("MongoDB Connection Error:", err));

// Import routes
const authorRoutes = require("./routes/author.routes");
const bookRoutes = require("./routes/book.routes");
const userRoutes = require("./routes/user.routes");
const transactionRoutes = require("./routes/transaction.routes");

// Use routes
app.use("/authors", authorRoutes);
app.use("/books", bookRoutes);
app.use("/users", userRoutes);
app.use("/transactions", transactionRoutes);

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));


author.model.js- 
const mongoose = require("mongoose");

const authorSchema = new mongoose.Schema({
  name: { type: String, required: true },
  birth_year: { type: Number, required: true },
  nationality: String,
  books: [{ type: mongoose.Schema.Types.ObjectId, ref: "Book" }],
});

const Author = mongoose.model("Author", authorSchema);
module.exports = Author;

book.model.js-
const mongoose = require("mongoose");

const bookSchema = new mongoose.Schema({
  title: { type: String, required: true, unique: true },
  published_year: Number,
  genre: String,
  author: { type: mongoose.Schema.Types.ObjectId, ref: "Author", required: true },
});

const Book = mongoose.model("Book", bookSchema);
module.exports = Book;

user.model.js-
const mongoose = require("mongoose");

const userSchema = new mongoose.Schema({
  username: { type: String, required: true, unique: true },
  email: { type: String, required: true, unique: true },
  borrowed_books: [{ type: mongoose.Schema.Types.ObjectId, ref: "Transaction" }],
});

const User = mongoose.model("User", userSchema);
module.exports = User;

transaction.model.js-
const mongoose = require("mongoose");

const transactionSchema = new mongoose.Schema({
  book: { type: mongoose.Schema.Types.ObjectId, ref: "Book", required: true },
  user: { type: mongoose.Schema.Types.ObjectId, ref: "User", required: true },
  borrow_date: { type: Date, default: Date.now },
  return_date: { type: Date, default: null },
});

const Transaction = mongoose.model("Transaction", transactionSchema);
module.exports = Transaction;

book.controller.js-
const Book = require("../models/book.model");

// Add a new book
const addBook = async (req, res) => {
  try {
    const newBook = new Book(req.body);
    await newBook.save();
    res.status(201).json(newBook);
  } catch (error) {
    res.status(500).json({ message: "Error adding book", error });
  }
};

// Get all books
const getAllBooks = async (req, res) => {
  try {
    const books = await Book.find().populate("author", "name");
    res.status(200).json(books);
  } catch (error) {
    res.status(500).json({ message: "Error fetching books", error });
  }
};

module.exports = { addBook, getAllBooks };
book.routes.js-
const express = require("express");
const { addBook, getAllBooks } = require("../controllers/book.controller");

const router = express.Router();

router.post("/", addBook);
router.get("/", getAllBooks);

module.exports = router;


transaction.controller.js
const Transaction = require("../models/transaction.model");
const Book = require("../models/book.model");

const borrowBook = async (req, res) => {
  const { bookId, userId } = req.body;
  
  try {
    const book = await Book.findById(bookId);
    if (!book) return res.status(404).json({ message: "Book not found" });

    const newTransaction = new Transaction({ book: bookId, user: userId });
    await newTransaction.save();

    res.status(201).json({ message: "Book borrowed successfully", newTransaction });
  } catch (error) {
    res.status(500).json({ message: "Error borrowing book", error });
  }
};

const returnBook = async (req, res) => {
  const { transactionId } = req.body;

  try {
    const transaction = await Transaction.findById(transactionId);
    if (!transaction) return res.status(404).json({ message: "Transaction not found" });

    transaction.return_date = new Date();
    await transaction.save();

    res.status(200).json({ message: "Book returned successfully", transaction });
  } catch (error) {
    res.status(500).json({ message: "Error returning book", error });
  }
};

module.exports = { borrowBook, returnBook };

transaction.routes.js-
const express = require("express");
const { borrowBook, returnBook } = require("../controllers/transaction.controller");

const router = express.Router();

router.post("/borrow", borrowBook);
router.post("/return", returnBook);

module.exports = router;
