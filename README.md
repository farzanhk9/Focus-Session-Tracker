import json
import time
from datetime import datetime
from pathlib import Path

DB_FILE = Path("focus_sessions.json")


class FocusTracker:

    def __init__(self):
        if not DB_FILE.exists():
            DB_FILE.write_text("[]")

    def load(self):
        with open(DB_FILE, "r") as f:
            return json.load(f)

    def save(self, data):
        with open(DB_FILE, "w") as f:
            json.dump(data, f, indent=4)

    def start_session(self):

        task = input("Task name: ")
        minutes = int(input("Focus duration (minutes): "))

        print(f"\nStarting {minutes} minute session...")
        print("Press Ctrl+C to stop.\n")

        start = datetime.now()

        try:
            for remaining in range(minutes * 60, 0, -1):

                mins = remaining // 60
                secs = remaining % 60

                print(
                    f"\rTime Left: {mins:02}:{secs:02}",
                    end=""
                )

                time.sleep(1)

        except KeyboardInterrupt:
            print("\nSession cancelled.")
            return

        end = datetime.now()

        data = self.load()

        data.append({
            "task": task,
            "start": start.isoformat(),
            "end": end.isoformat(),
            "duration": minutes
        })

        self.save(data)

        print("\n\nSession completed!")

    def statistics(self):

        data = self.load()

        if not data:
            print("No sessions found.")
            return

        total_time = sum(
            session["duration"]
            for session in data
        )

        total_sessions = len(data)

        print("\n=== Statistics ===")
        print(f"Sessions: {total_sessions}")
        print(f"Focused Time: {total_time} min")
        print(
            f"Average Session: "
            f"{round(total_time/total_sessions, 2)} min"
        )

        print("\nRecent Sessions:\n")

        for session in data[-5:]:

            print(
                f"{session['task']} "
                f"({session['duration']} min)"
            )

    def streak(self):

        data = self.load()

        dates = sorted(
            {
                session["start"][:10]
                for session in data
            }
        )

        print(
            f"\nDays with focus sessions: "
            f"{len(dates)}"
        )


def main():

    tracker = FocusTracker()

    while True:

        print("\n=== Focus Tracker ===")
        print("1. Start Session")
        print("2. Statistics")
        print("3. Streak")
        print("4. Exit")

        choice = input("\nSelect: ")

        if choice == "1":
            tracker.start_session()

        elif choice == "2":
            tracker.statistics()

        elif choice == "3":
            tracker.streak()

        elif choice == "4":
            break

        else:
            print("Invalid option")


if __name__ == "__main__":
    main()
