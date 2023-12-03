import React, { useEffect, useState } from 'react';
import cardData from "./cardData";
import { swipefunction } from "./swipeUtils";
import Button from "./SwipeButton";
import { entitiesApi } from "../api/entitiesApi";
import { imagesApi } from "../api/imagesApi";
import DislikeButton from "./DislikeButton";
import MaybeButton from "./MaybeButton";
import LikeButton from "./LikeButton";
import { HiOutlineInformationCircle } from "react-icons/hi";

/**
 * React functional component for handling group swiping actions, including card swiping, button interactions,
 * and displaying match results.
 *
 * @component
 * @param {Object} props - The component properties.
 * @param {SocketIO.Socket} props.socket - The Socket.IO socket instance.
 * @param {string} props.username - The username of the current user.
 * @param {string} props.room - The room identifier for the group swipe session.
 * @example
 * // Example usage within another React component
 * import GroupSwipe from './GroupSwipe';
 * //...
 * <GroupSwipe socket={socket} username="JohnDoe" room="group123" />
 */
function GroupSwipe({ socket, username, room }) {
    /**
     * State to manage the array of cards used for swiping.
     * @type {Object[]}
     */
    const [cards, setCards] = useState(cardData);

    /**
     * State to manage the index of the current card being displayed.
     * @type {number}
     */
    const [currentCard, setCurrentCard] = useState(0);

    /**
     * State to store the array of liked cards.
     * @type {Object[]}
     */
    const [likedCards, setLikedCards] = useState([]);

    /**
     * State to store the array of disliked cards.
     * @type {Object[]}
     */
    const [dislikedCards, setDislikedCards] = useState([]);

    /**
     * State to store the array of maybe cards.
     * @type {Object[]}
     */
    const [maybeCards, setMaybeCards] = useState([]);

    /**
     * State to indicate whether the component is waiting for match results.
     * @type {boolean}
     */
    const [waiting, setWaiting] = useState(false);

    /**
     * State to store the formatted result of matched cards.
     * @type {JSX.Element[] | string}
     */
    const [result, setResult] = useState('');

    /**
     * State to control the visibility of the card description modal.
     * @type {boolean}
     */
    const [showDescription, setShowDescription] = useState(false);

    /**
     * State to store the total number of cards in the session.
     * @type {number}
     */
    const [totalCards, setTotalCards] = useState(0);

    /**
     * State to store the index of the current swipe action.
     * @type {number}
     */
    const [swipeIndex, setSwipeIndex] = useState(0);

    /**
     * Log the current cards array to the console.
     */
    console.log(cards);

    /**
     * Variable to store the starting X-coordinate during drag.
     * @type {number}
     */
    let startX = 0;

    /**
     * Effect hook to fetch movie data when the component mounts.
     */
    useEffect(() => {
        fetchMovieData();
    }, []);

    /**
     * Effect hook to set up event listeners for match results and cleanup on unmount.
     */
    useEffect(() => {
        if (cards.length === 0) {
            handleChooseMovie();
        }

        socket.on('match_result', (matchedCardsWithScores) => {
            const formattedResult = matchedCardsWithScores
                .slice(0, 3)
                .map((card, index) => (
                    <li key={index}>
                        <div className="resultPoster">
                            <img
                                src={card.poster}
                                alt={`Card ${card.id}`}
                            />
                            <div className="movieTitle">
                                <span className="idNumber">#{index + 1}.</span> {card.title}
                            </div>
                        </div>
                    </li>
                ));
            setResult(formattedResult);
            setWaiting(false);
        });

        return () => {
            socket.off('match_result');
        };
    }, [cards]);

    /**
     * Handles the swipe action based on the drag direction.
     *
     * @param {string} direction - The direction of the swipe ("left" or "right").
     */
    const swipe = (direction) => {
        swipefunction(
            direction,
            cards,
            currentCard,
            setCards,
            setLikedCards,
            setDislikedCards,
        );
        setSwipeIndex((prevIndex) => prevIndex + 1);
    };

    /**
     * Handles the start of a drag action.
     *
     * @param {React.MouseEvent} e - The drag start event.
     */
    const handleDragStart = (e) => {
        startX = e.clientX;
    };

    /**
     * Handles the end of a drag action, determining the drag direction and triggering a swipe.
     *
     * @param {React.MouseEvent} e - The drag end event.
     */
    const handleDragEnd = (e) => {
        const dragDirection = e.clientX - startX < 0 ? "left" : "right";
        if (dragDirection === "left") {
            swipe("left");
        } else if (dragDirection === "right") {
            swipe("right");
        }
    };

    /**
     * Moves the current card to the "Maybe" category.
     */
    const maybe = () => {
        const updatedMaybe = [...maybeCards, { ...cards[currentCard] }];
        setMaybeCards(updatedMaybe);
        setCards((prevCards) =>
            prevCards.filter((_, index) => index !== currentCard)
        );
    };

    /**
     * Handles the end of the group swipe session, triggering the server to process match results.
     */
    const handleChooseMovie = () => {
        if (likedCards !== null && room !== '') {
            socket.emit('reset_scores', { room });
            socket.emit('choose_movie', { likedCards, dislikedCards, maybeCards, room });
            setWaiting(true);
        }
    };

    /**
     * Fetches movie data from the server and updates the cards state.
     */
    const fetchMovieData = async () => {
        try {
            const movieData = await entitiesApi.getMovies();
            const posterPromises = movieData.map(movie => fetchMoviePoster(movie.id));
            const posterDataArray = await Promise.all(posterPromises);
            const updatedCardData = movieData.map((movie, index) => ({
                id: movie.id,
                image: URL.createObjectURL(new Blob([posterDataArray[index]])),
                title: movie.title,
                rated: movie.rated,
                description: movie.description,
                year: movie.year,
            }));
            setCards(updatedCardData);
            setTotalCards(updatedCardData.length);
        } catch (error) {
            console.error('Error fetching movie data:', error);
        }
    };

    /**
     * Fetches the movie poster for a given movie ID.
     *
     * @param {number} movieId - The ID of the movie.
     * @returns {Promise<Blob>} - The movie poster data.
     */
    const fetchMoviePoster = async (movieId) => {
        try {
            return await imagesApi.getMoviePosterById(movieId);
        } catch (error) {
            console.error('Error fetching movie poster:', error);
        }
    };

    /**
     * Shows the description modal for the current card.
     */
    const showInfo = () => {
        setShowDescription(true);
    };

    /**
     * Hides the description modal.
     */
    const hideInfo = () => {
        setShowDescription(false);
    };

    /**
     * Renders the GroupSwipe component.
     *
     * @returns {JSX.Element} - JSX representation of the GroupSwipe component.
     */
    return (
        <div>
            <div className="cardArea">
                {cards.length > 0 ? (
                    <div className="cardContainer"
                         draggable
                         onDragStart={handleDragStart}
                         onDragEnd={handleDragEnd}
                    >
                        <div className="posterContainer">
                            <img className="posterImage"
                                 src={cards[currentCard].image}
                                 alt={`Card ${cards[currentCard].id}`}
                            />
                            <div className="darkOverlay">
                                <h2>{cards[currentCard].title} / {cards[currentCard].rated}</h2>
                            </div>
                            {showDescription && (
                                <div className="overlay" onClick={hideInfo}>
                                    <div className="descriptionModal">
                                        <h2>{cards[currentCard].title}</h2>
                                        <p>{cards[currentCard].description}</p>
                                    </div>
                                </div>
                            )}
                            <div className="darkOverlay2">
                                <p className="counter">{swipeIndex + 1}/{totalCards}</p>
                                <HiOutlineInformationCircle className="info" onClick={showInfo} />
                            </div>
                        </div>
                        <div className="swipeButtons row align-self-center">
                            <div className="dis-button col">
                                <DislikeButton
                                    onClick={() => swipe("left")}
                                    text=""
                                    disabled={cards.length === 0}
                                />
                            </div>
                            <div className="may-button col">
                                <MaybeButton
                                    onClick={maybe}
                                    text=""
                                    disabled={cards.length === 0}
                                />
                            </div>
                            <div className="lik-button col">
                                <LikeButton
                                    onClick={() => swipe("right")}
                                    text=""
                                    disabled={cards.length === 0}
                                />
                            </div>
                        </div>
                    </div>
                ) : (
                    <div className="cm-form result">
                        {waiting && <p>Wait for other users to finish...</p>}
                        {result && (
                            <React.Fragment>
                                <h1>Your Top 3!</h1>
                                <ul className="result-list">{result}</ul>
                            </React.Fragment>
                        )}
                    </div>
                )}
            </div>
        </div>
    );
}

/**
 * Default export of the GroupSwipe component.
 * @exports GroupSwipe
 */
export default GroupSwipe;
