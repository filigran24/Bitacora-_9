# Codigo JSX
// CardsRecr.js
import React, {
  useState,
  useMemo,
  useRef,
  useEffect,
  useCallback,
} from "react";
import { Droppable, Draggable } from "react-beautiful-dnd";
import "./CardsRecr.css";

// Asegúrate de que esta ruta sea correcta para tu componente Modal.
// He asumido que el nombre del archivo es ModalDetalleCardsMKT_RECR.js
import ModalDetalleCardsMKT_RECR from "../ModalCopmponent/ModalDetalleCardsMKT_RECR";

import { Tooltip } from "react-tooltip";

const COLORS_ARRAY = [
  "#98b46d",
  "#00b4b4",
  "#ff7f50",
  "#F4D03F",
  "#A3D8A3",
  "#F9E79F",
  "#A9CCE3",
  "#D7BDE2",
];

const getInitials = (name) => {
  if (!name || typeof name !== "string" || name.trim() === "") {
    return "??";
  }
  const parts = name.trim().split(/\s+/).filter(Boolean);
  if (parts.length === 0) return "??";
  if (parts.length === 1) return parts[0].substring(0, 2).toUpperCase();
  return (parts[0][0] + parts[parts.length - 1][0]).toUpperCase();
};

const getUserColor = (name, users) => {
  const index = users.findIndex((user) => user.name === name);
  if (index === -1) {
    return "#808080";
  }
  return COLORS_ARRAY[index % COLORS_ARRAY.length];
};

const formatTimeElapsed = (creationDate) => {
  const created = new Date(creationDate);
  if (isNaN(created.getTime())) {
    return "Invalid Date";
  }
  const now = new Date();
  const diffTime = Math.abs(now.getTime() - created.getTime());
  const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24));
  return diffDays;
};

// =========================================================================
// Componente de Tarjeta
// Recibe la nueva prop `onCardClick`
// =========================================================================
const RecrCard = ({
  user,
  provided,
  recruiterUsers,
  handleUserUpdate,
  onCardClick,
}) => {
  const [isSelectorOpen, setIsSelectorOpen] = useState(false);
  const [recruiterFilter, setRecruiterFilter] = useState("");
  const selectorRef = useRef(null);

  const assignedRecruiterUser = useMemo(() => {
    if (user.assignedRecruiterId) {
      return recruiterUsers.find((rec) => rec.id === user.assignedRecruiterId);
    }
    return null;
  }, [user.assignedRecruiterId, recruiterUsers]);

  const initials = assignedRecruiterUser
    ? getInitials(assignedRecruiterUser.name)
    : getInitials(user.name) || "US";

  const color = assignedRecruiterUser
    ? getUserColor(assignedRecruiterUser.name, recruiterUsers)
    : getUserColor(user.name, recruiterUsers);

  const showTimeBadge =
    user.status === "In Progress" || user.status === "First Contact";

  const handleRecruiterInitialsClick = useCallback((e) => {
    e.stopPropagation();
    setIsSelectorOpen((prev) => !prev);
    setRecruiterFilter("");
  }, []);

  const handleUserSelect = useCallback(
    (e, selectedUser) => {
      // Check if the event object exists before trying to stop propagation
      if (e) {
        e.stopPropagation();
      }
      handleUserUpdate(user.id, selectedUser);
      setIsSelectorOpen(false);
      setRecruiterFilter("");
    },
    [handleUserUpdate, user.id]
  );

  useEffect(() => {
    const handleClickOutside = (event) => {
      if (selectorRef.current && !selectorRef.current.contains(event.target)) {
        setIsSelectorOpen(false);
        setRecruiterFilter("");
      }
    };
    document.addEventListener("mousedown", handleClickOutside);
    return () => {
      document.removeEventListener("mousedown", handleClickOutside);
    };
  }, []);

  const filteredRecruiterUsers = useMemo(() => {
    if (!recruiterFilter) {
      return recruiterUsers;
    }
    const lowercasedFilter = recruiterFilter.toLowerCase();
    return recruiterUsers.filter((u) =>
      u.name.toLowerCase().includes(lowercasedFilter)
    );
  }, [recruiterUsers, recruiterFilter]);

  return (
    <div
      className={`movable-item ${user.status
        .toLowerCase()
        .replace(/\s/g, "-")}-card`}
      ref={provided.innerRef}
      {...provided.draggableProps}
      {...provided.dragHandleProps}
      onClick={() => onCardClick(user)} // Agrega el evento de clic aquí
    >
      <div className="custom-card-content">
        <div className="card-heade">
          <h3 className="card-name">{user.name}</h3>
        </div>

        {user.referredBy && (
          <div className="card-referred-by">
            <span className="label">Referred by:</span>
            <span className="value"> {user.referredBy}</span>
          </div>
        )}

        <div className="card-foote">
          <p className="card-dat">{user.date}</p>
          {showTimeBadge ? (
            <div className="in-progress-badge">
              <span role="img" aria-label="clock"></span>
              <span className="card-met">
                {formatTimeElapsed(user.creationDate)}
              </span>
            </div>
          ) : (
            <span className="card-met">
              {formatTimeElapsed(user.creationDate)}
            </span>
          )}
        </div>

        {/* Selector de Reclutadores */}
        <div className="card-initials-selector-container">
          <div
            className="card-initials-display"
            style={{ backgroundColor: color }}
            onClick={handleRecruiterInitialsClick}
            data-tooltip-id={`recruiter-tooltip-${user.id}`}
            data-tooltip-content={
              assignedRecruiterUser
                ? assignedRecruiterUser.name
                : "Select Recruiter"
            }
            data-tooltip-place="bottom"
          >
            <h6>{initials}</h6>
          </div>
          <Tooltip id={`recruiter-tooltip-${user.id}`} />

          {isSelectorOpen && (
            <div
              style={{ width: "30px", right: "-20px" }}
              className="mkt-users-dropdown"
              ref={selectorRef}
            >
              <input
                style={{ width: "150px", position: "relative", right: "-10px" }}
                type="text"
                placeholder="Filter recruiters..."
                value={recruiterFilter}
                onChange={(e) => setRecruiterFilter(e.target.value)}
                onClick={(e) => e.stopPropagation()}
              />
              <ul>
                {filteredRecruiterUsers.length > 0 ? (
                  filteredRecruiterUsers.map((recruiterUser) => (
                    <li
                      key={recruiterUser.id}
                      onClick={(e) => handleUserSelect(e, recruiterUser)}
                      className={
                        assignedRecruiterUser &&
                        assignedRecruiterUser.id === recruiterUser.id
                          ? "selected"
                          : ""
                      }
                    >
                      <div
                        className="mkt-user-option-initials"
                        style={{
                          backgroundColor: getUserColor(
                            recruiterUser.name,
                            recruiterUsers
                          ),
                        }}
                      >
                        {getInitials(recruiterUser.name)}
                      </div>
                      <span className="mkt-user-option-name">
                        {recruiterUser.name}
                      </span>
                    </li>
                  ))
                ) : (
                  <div className="no-results-message">No recruiters found</div>
                )}
              </ul>
            </div>
          )}
        </div>
      </div>
    </div>
  );
};

// =========================================================================
// Main Component
// =========================================================================
function CardRecr({
  inProgressUsers,
  firstContactUsers,
  taInterviewUsers,
  noResponseUsers,
  rejectedUsers,
  hiredUsers,
  handleUserUpdate,
  recruiterUsers,
}) {
  // Estado para la ventana modal
  const [selectedCard, setSelectedCard] = useState(null);

  // Funciones para manejar la modal
  const handleCardClick = useCallback((card) => {
    setSelectedCard(card);
  }, []);

  const handleCloseModal = useCallback(() => {
    setSelectedCard(null);
  }, []);

  const renderCards = useCallback(
    (users) => {
      console.log(
        "Rendering cards. IDs:",
        users.map((u) => u.id)
      );
      if (!users || users.length === 0) {
        return <p className="no-cards-message"></p>;
      }

      return users.map((user, index) => (
        <Draggable key={user.id} draggableId={user.id} index={index}>
          {(provided) => (
            <RecrCard
              user={user}
              provided={provided}
              recruiterUsers={recruiterUsers}
              handleUserUpdate={handleUserUpdate}
              onCardClick={handleCardClick} // Pasa la función para abrir la modal
            />
          )}
        </Draggable>
      ));
    },
    [recruiterUsers, handleUserUpdate, handleCardClick]
  );

  const renderColumn = (columnId, title, users) => (
    <Droppable droppableId={columnId}>
      {(provided, snapshot) => (
        <div
          className={`cards-8 column card ${
            snapshot.isDraggingOver ? "dragging-over" : ""
          }`}
          {...provided.droppableProps}
          ref={provided.innerRef}
        >
          <h2 className="column-title">{title}</h2>
          <hr className="hr" />
          {renderCards(users)}
          {provided.placeholder}
        </div>
      )}
    </Droppable>
  );

  return (
    <div className="contenedor">
      {renderColumn("in-progress", "In Progress", inProgressUsers)}
      {renderColumn("first-contact", "First Contact", firstContactUsers)}
      {renderColumn("ta-interview", "TA Interview", taInterviewUsers)}
      {renderColumn("no-response", "No Response", noResponseUsers)}
      {renderColumn("rejected", "Rejected", rejectedUsers)}
      {renderColumn("hired", "Hired", hiredUsers)}

      {/* Renderizado condicional del componente Modal */}
      {selectedCard && (
        // eslint-disable-next-line react/jsx-pascal-case
        <ModalDetalleCardsMKT_RECR
          card={selectedCard}
          onClose={handleCloseModal}
        />
      )}
    </div>
  );
}

export default CardRecr;

#Codigo Css

/* src/components/CardsRecrComponent/CardsRecr.css */

.contenedor {
  display: flex;
  flex-wrap: nowrap;
  gap: 10px;
  margin-top: 10px;
  margin-bottom: 20px;
  padding: 0 10px;
  position: relative;
}

.column-title {
  display: flex;
  align-items: center;
  justify-content: flex-start;
  width: 100%;
  position: relative;
  font-weight: bold;
  color: #333;
  margin-bottom: 15px;
  font-size: 1.2em;
}

/* Tus estilos de columna existentes */

/* La clase que maneja el efecto de borde azul */
.cards-8.column.card.dragging-over {
  border: dashed 2px #007bff;
  background-color: #e9f0f8;
}

.hr {
  border: none;
  height: 1px;
  background-color: #bbb;
}
.card-initials-display {
  position: absolute;
  bottom: 0;
  right: 0;
}

.cards-8 {
  display: flex;
  flex-direction: column;
  width: 100%;
  height: 60vh;
  background-color: #fff;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  padding: 15px;
  box-sizing: border-box;
}

/* LOS SIGUIENTES ESTILOS SON OBSOLETOS SI ESTÁS USANDO LAS CLASES DE 'cards.css'
   PARA EL DISEÑO DE LA TARJETA INDIVIDUAL (ej. .movable-item, .custom-card-content, etc.)
   Por lo tanto, los he comentado o eliminado.
*/
/*
.recr-card {
    display: flex;
    flex-direction: column;
    position: relative;
    top: 20px;
}
.countDias {
    position: absolute;
    top: 170px;
    right: 60px;
}
.Date {
    font-size: 12px;
}
.referedBy {
    font-size: 12px;
    color: #555;
}
.referente {
    font-size: 15px;
    color: #000;
}
.recr-initials {
    position: absolute;
    bottom: 0px;
    right: 5px;
}
*/

/* Media Query para Laptops (ej: min-width de 768px - para 2 o 3 columnas) */
@media (min-width: 768px) {
  .contenedor {
    gap: 15px;
  }

  .cards-8 {
    flex-basis: calc(33.33% - 10px);
  }
}

/* Media Query para Pantallas Grandes (ej: min-width de 1200px - para 4 o 5 columnas) */
@media (min-width: 1200px) {
  .cards-8 {
    flex-basis: calc(16.66% - 16.66px); /* Para 6 columnas */

    margin-top: 30px;
  }
}

@media (min-width: 1600px) {
  .cards-8 {
    flex-basis: calc(16.66% - 16.66px);
  }
}

<img width="1364" height="630" alt="image" src="https://github.com/user-attachments/assets/6be0ee36-f193-4bd4-be4b-4442e0627a34" />


