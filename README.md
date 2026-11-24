# Main module controlling the Disneyland Review Analyser.

import process
import tui
import visual
import exporter

def handle_view_data(reviews):
    """Handle the View Data menu."""

    choice = tui.display_view_data_menu()

    if choice is None:
        return
    menu_options = {
        "A": "View Reviews by Park",
        "B": "Number of Reviews by Park and Reviewer Location",
        "C": "Average Score per year by Park",
        "D": "Average Score per Park by Reviewer Location"
    }

    tui.display_message(f"You have chosen option {choice} - {menu_options[choice]}")

    parks = process.get_parks(reviews)

    if choice == "A":
        selected_park = tui.get_park_choice(parks)

        if selected_park is not None:
            park_reviews = process.get_reviews_by_park(
                reviews, selected_park
            )

            tui.display_reviews(park_reviews, selected_park)

    elif choice == "B":
        selected_park = tui.get_park_choice(parks)

        if selected_park is not None:
            locations = process.get_locations(reviews)
            selected_location = tui.get_location_choice(locations)

            if selected_location is not None:
                review_count = (
                    process.get_review_count_by_park_and_location(
                        reviews,
                        selected_park,
                        selected_location
                    )
                )

                tui.display_review_count(
                    selected_park,
                    selected_location,
                    review_count
                )

    elif choice == "C":
        selected_park = tui.get_park_choice(parks)

        if selected_park is not None:
            years = process.get_years_by_park(
                reviews, selected_park
            )

            selected_year = tui.get_year_choice(years)

            if selected_year is not None:
                average_score = process.get_average_score_by_year(
                    reviews,
                    selected_park,
                    selected_year
                )

                tui.display_average_score(
                    selected_park,
                    selected_year,
                    average_score
                )

    elif choice == "D":
        results = process.get_average_rating_per_park_location(
            reviews
        )

        tui.display_average_rating_per_park_location(results)

def handle_visualise_data(reviews):
    """Handle the Visualise Data menu."""
    choice = tui.display_visualise_data_menu()

    if choice is None:
        return

    menu_options = {
        "A": "Most reviewed Parks",
        "B": "Park Ranking by Nationality",
        "C": "Most Popular Month by Park"
    }

    tui.display_message(
        f"You have chosen option {choice} - {menu_options[choice]}"
    )

    if choice == "A":
        review_counts = process.get_review_counts_by_park(reviews)
        visual.plot_most_reviewed_parks(review_counts)

    elif choice == "B":
        parks = process.get_parks(reviews)
        selected_park = tui.get_park_choice(parks)

        if selected_park is not None:
            location_averages = (
                process.get_average_rating_by_location(
                    reviews, selected_park
                )
            )
            print("\nTop 10 locations and their average ratings:")

            for location, average in location_averages:
                print(f"{location}: {average}")

            visual.plot_top_locations_by_average_rating(
                location_averages, selected_park
            )

    elif choice == "C":
        parks = process.get_parks(reviews)
        selected_park = tui.get_park_choice(parks)

        if selected_park is not None:
            monthly_averages = (
                process.get_average_rating_by_month(
                    reviews, selected_park
                )
            )

            visual.plot_average_rating_by_month(
                monthly_averages, selected_park
            )

def handle_export_data(reviews):
    """Handle the export data feature."""
    selected_format = tui.get_export_format()

    data_exporter = exporter.ParkDataExporter(reviews)

    file_path = data_exporter.export(selected_format)

    tui.display_message(
        f"Data successfully exported to: {file_path}"
    )

def main():
    """Run the Disneyland Review Analyser application."""

    tui.display_title()

    try:
        reviews = process.load_data()

    except FileNotFoundError:
        tui.display_message(
            "Error: Disneyland reviews dataset could not be found."
        )
        return

    except (ValueError, KeyError) as error:
        tui.display_message(
            f"Error while reading the dataset: {error}"
        )
        return

    tui.display_message("Dataset successfully loaded.")
    tui.display_message(
        f"Number of reviews loaded: {len(reviews)}"
    )

    running = True

    while running:

        tui.display_main_menu()
        choice = tui.get_main_menu_choice()

        if choice == "A":
            tui.display_message(
                "You have chosen option A - View Data"
            )
            handle_view_data(reviews)

        elif choice == "B":
            tui.display_message(
                "You have chosen option B - Visualise Data"
            )
            handle_visualise_data(reviews)

        elif choice == "C":
            tui.display_message(
                "You have chosen option C - Export Data"
            )
            handle_export_data(reviews)

        elif choice == "X":
            tui.display_message(
                "You have chosen option X - Exit"
            )
            tui.display_message(
                "Thank you for using Disneyland Review Analyser."
            )
            running = False

if __name__ == "__main__":
    main()
