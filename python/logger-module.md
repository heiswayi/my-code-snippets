Reusable logger module for Python app development. This is part of my `utils` modules.

logger.py
```python
import logging
import os
from datetime import datetime

class Logger:
    def __init__(
        self,
        name="Application",
        log_folder="logs",
        level=logging.INFO,
        date_format="%Y-%m-%d %H:%M:%S"
    ):
        self.name = name
        self.log_folder = log_folder
        self.level = level
        self.date_format = date_format
        self.logger = logging.getLogger(name)
        self.logger.setLevel(level)
        self.logger.propagate = False  # Avoid duplicate log entries
        
        # Ensure log folder exists
        if not os.path.exists(log_folder):
            os.makedirs(log_folder)
        
        # Create a handler for file logging
        log_filename = self.get_log_file_path()
        file_handler = logging.FileHandler(log_filename)
        file_handler.setLevel(level)

        # Define the log format
        log_format = f"%(asctime)s %(levelname)s [%(name)s] %(message)s [%(filename)s:%(lineno)d]"
        formatter = logging.Formatter(log_format, datefmt=self.date_format)
        file_handler.setFormatter(formatter)
        
        # Create a handler for console logging
        console_handler = logging.StreamHandler()
        console_handler.setLevel(self.level)
        console_handler.setFormatter(formatter)

        # Remove existing handlers to avoid duplicate logs
        if self.logger.hasHandlers():
            self.logger.handlers.clear()

        # Add both file and console handlers to the logger
        self.logger.addHandler(file_handler)
        self.logger.addHandler(console_handler)
        
    def get_log_file_path(self):
        """Generate a log file path with the current date."""
        date_str = datetime.now().strftime("%Y-%m-%d")
        return os.path.join(self.log_folder, f"{date_str}.log")

    def get_logger(self):
        return self.logger

# # Example usage
# if __name__ == "__main__":
#     # Initialize the logger with custom settings
#     my_logger = Logger(
#         name="MyCustomLogger",
#         log_folder="my_logs",
#         level=logging.DEBUG,
#         date_format="%Y-%m-%d %H:%M:%S"
#     ).get_logger()
    
#     # Log some messages
#     my_logger.debug("This is a debug message.")
#     my_logger.info("This is an info message.")
#     my_logger.warning("This is a warning message.")
#     my_logger.error("This is an error message.")
#     my_logger.critical("This is a critical message.")
```
